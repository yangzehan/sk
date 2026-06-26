---
title: auto-consolidate 子进程设计
type: reference
created: 2026-06-26
updated: 2026-06-26
tags: [pi, pi-hermes-memory, auto-consolidate, 子进程, fifo-evict, reject, 容量溢出, pi -p, retry]
related:
  - "[[INDEX]]"
  - "[[概览]]"
  - "[[系统提示词注入]]"
  - "[[后台机制 - 三大处理器]]"
---

# auto-consolidate 子进程设计

> 回答两个问题：
> 1. auto-consolidate 模式下，扩展怎么解决"memory 满了"？
> 2. 另两个策略 `reject` / `fifo-evict` 怎么执行？

---

## 入口与触发

### 策略字段（`MemoryConfig`）

```ts
memoryOverflowStrategy?: "auto-consolidate" | "reject" | "fifo-evict";  // 默认 auto-consolidate
autoConsolidate: boolean;                                              // 默认 true（旧 alias）
```

两者兼容（`config.ts:131-135`）：

| 同时设置 | 结果 |
|---|---|
| `memoryOverflowStrategy: "fifo-evict"` + `autoConsolidate: true` | `memoryOverflowStrategy` 胜出，`autoConsolidate` 同步为 `true` |
| 只设 `autoConsolidate: false` | 映射成 `memoryOverflowStrategy: "reject"` |
| 只设 `memoryOverflowStrategy: "auto-consolidate"` | `autoConsolidate` 同步为 `true` |
| 都不设 | 默认 `"auto-consolidate"` |

### 触发点：`MemoryStore._add()`（`memory-store.ts:165-188`）

```ts
const newTotal = [...entries, encoded].join(ENTRY_DELIMITER).length;
if (newTotal > limit) {
  const strategy = this.memoryOverflowStrategy();

  if (strategy === "fifo-evict") {
    return this.fifoEvictAndAdd(target, entries, encoded, content.length, limit);
  }

  // Auto-consolidate once if configured — limit retries to prevent infinite loops
  if (strategy === "auto-consolidate" && this.consolidator && _retriesLeft > 0) {
    try {
      const result = await this.consolidator(target, signal);  // → triggerConsolidation()
      if (result.consolidated) {
        await this.loadFromDisk();    // ← CRITICAL：子进程改了文件，本地数组过期
        return this._add(target, content, signal, _retriesLeft - 1, addedMessage);
      }
    } catch { /* fall through */ }
  }
  return this.memoryFullError(target, content.length);
}
```

→ 关键：三种策略都只在 **写入** 时触发（`add` 操作），不会主动后台跑。

---

## 子进程命令行构造

子进程由 `triggerConsolidation()` 调 `execChildPrompt()` 启动。

### 命令行格式（`pi-child-process.ts:90-99`）

```ts
const args = ["-p", "--no-session"];
if (model)    args.push("--model", model);
if (thinking) args.push("--thinking", thinking);
appendOwnExtensionArgs(args);  // --no-extensions + -e OWN_EXTENSION_PATH
args.push(prompt);
```

最终命令行形如：

```bash
pi -p --no-session \
   --model anthropic/claude-opus-4-7 \
   --thinking high \
   --no-extensions \
   -e /home/yzh/.pi/agent/npm/node_modules/pi-hermes-memory/src/index.ts \
   "<CONSOLIDATION_PROMPT 全文 + 当前 entries + 'Use the memory tool to consolidate. Target: \"<toolTarget>\"'>"
```

### 关键开关

| 开关 | 干什么 | 为什么 |
|---|---|---|
| `-p` | non-interactive print mode | 不开交互 UI |
| `--no-session` | 不写 session 文件 | 临时任务，不污染主 session 历史 |
| `--no-extensions` | 跳过 `settings.json` 里所有 pi-package | 不加载 context-mode、pi-lens、pi-web-access 等无关扩展 |
| `-e <OWN_EXTENSION_PATH>` | **只**加载 pi-hermes-memory 自己的 `src/index.ts` | 子进程需要 `memory` 工具 |

`OWN_EXTENSION_PATH` 通过 `import.meta.url` 算出（`pi-child-process.ts:30-38`）：

```ts
const OWN_EXTENSION_PATH = resolve(dirname(fileURLToPath(import.meta.url)), "../index.ts");
```

**Windows 兼容**（`pi-child-process.ts:106-120`）：

```ts
if (platform === "win32") {
  // 用 process.execPath（node）直接跑 cli.js，不用 pi shim
  return { command: process.execPath, args: [piCliPath, ...args] };
}
```

---

## 子进程能用什么工具

子进程**只**加载 `pi-hermes-memory` 自己，所以它能用：

- ✅ `memory`（add / replace / remove 三个 action）
- ✅ `memory_search`（FTS5 搜索）
- ✅ `session_search`
- ✅ `skill_manage`
- ❌ 主进程的 `read` / `bash` / `edit` / `write` / 其他扩展工具（**故意禁掉**）

子进程用 `memory` 工具直接写 MEMORY.md / USER.md → 落盘 → 退码 0 → 主进程收 `{ consolidated: true }`。

---

## 子进程拿到的 Prompt（`auto-consolidate.ts:62-72`）

```ts
const prompt = [
  CONSOLIDATION_PROMPT,                                // 见下
  "",
  `--- Current ${labelForTarget(target, toolTarget)} Entries ---`,
  currentContent || "(empty)",
  "",
  `Use the memory tool to consolidate. Target: '${toolTarget}'`,
].join("\n");
```

### `CONSOLIDATION_PROMPT` 全文（`constants.ts:152-163`）

```
The memory is at capacity. Review the current entries and consolidate them:
- Merge related entries into a single, concise entry
- Remove outdated or superseded entries (entries older than 30 days without recent references are candidates for removal)
- Keep the most important and frequently-referenced facts
- Preserve user preferences and corrections (highest priority)

Each entry shows when it was created and last referenced in HTML comments (<!-- created=..., last=... -->). Use this to identify stale entries.

Use the memory tool to make changes. Be aggressive about merging — less is more.
```

---

## Retry 策略（`pi-child-process.ts:135-166`）

```ts
const result = await pi.exec(invocation.command, invocation.args, execOptions);
if (
  result.code === 0 ||                              // 成功
  !options.retryWithoutOverrides ||                  // 没开 retry
  !hasChildLlmOverrides(config) ||                  // 配置无 override
  !shouldRetryWithoutOverrides(result)              // 错误不属于"override 问题"
) {
  return result;
}
// 失败 + 开了 retry + 有 override + 错误像是 override 引起的
const retryInvocation = resolveChildPiInvocation(basePromptArgs(prompt));  // 不带 model/thinking
return pi.exec(retryInvocation.command, retryInvocation.args, execOptions);
```

### "override 失败"识别规则（`pi-child-process.ts:52-54, 132`）

```ts
const OVERRIDE_FAILURE_SUBJECT = /\b(model|provider|thinking)\b/i;
const OVERRIDE_FAILURE_REASON  = /\b(not found|unknown|invalid|unsupported|unavailable|unrecognized|no match|no matches|cannot resolve|failed to resolve)\b/i;
```

→ stderr/stdout **同时**出现 `model|provider|thinking` **和** `not found|invalid|...` 才算"override 失败"。

**典型场景**：你设了 `llmModelOverride: "anthropic/claude-opus-4-7"`，但当前 provider 没这个模型 → 子进程报错 → 自动剥掉 `--model` `--thinking` 重试一次。

---

## 主进程必须 reload

子进程返回成功后（`triggerConsolidation` 返回 `{ consolidated: true }`），调用方（`memory-store.ts:178`）**强制**执行：

```ts
if (result.consolidated) {
  await this.loadFromDisk();    // ← CRITICAL：子进程改了文件，本地数组过期
  return this._add(target, content, signal, _retriesLeft - 1, addedMessage);  // 重试 1 次
}
```

注释里大写 `CRITICAL`，是反复踩过的坑。

### 重试限制

`_retriesLeft - 1` → 0，**只重试一次**（防止无限循环）。如果重试后还超 limit，就 fallback 到 `memoryFullError()`。

---

## 三种策略详解

### ① `fifo-evict`（先入先出淘汰）

`memory-store.ts:168-169` → `fifoEvictAndAdd()`（line 196-225）：

```ts
const remaining = [...entries];
const evictedEntries: string[] = [];

while ([...remaining, encoded].join(ENTRY_DELIMITER).length > limit && remaining.length > 0) {
  const evicted = remaining.shift()!;  // ← 从头部扔掉
  evictedEntries.push(this.stripMetadata(evicted));
}

remaining.push(encoded);
this.setEntries(target, remaining);
await this.saveToDisk(target);

return {
  ...this.successResponse(target, "Memory updated. Rotated N older entries..."),
  evicted_entries: evictedEntries,
  evicted_count: evictedEntries.length,
};
```

- **同步执行**，不调 LLM
- 一直 `shift()` 头部（最早写入的）直到能装下新条目
- 返回 `evicted_entries` 数组（stripped 后的内容）
- **特例**：单条新内容**本身**就超过 limit → 直接 `memoryFullError()` 拒收

### ② `auto-consolidate`（自动调 LLM 合并）

`memory-store.ts:175-187`：

```ts
if (strategy === "auto-consolidate" && this.consolidator && _retriesLeft > 0) {
  try {
    const result = await this.consolidator(target, signal);   // → triggerConsolidation()
    if (result.consolidated) {
      await this.loadFromDisk();
      return this._add(target, content, signal, _retriesLeft - 1, addedMessage);
    }
  } catch { /* fall through */ }
}
return this.memoryFullError(target, content.length);
```

- **consolidator 是什么**：在 `index.ts:191-205` 注入：
  ```ts
  store.setConsolidator(async (target, signal) => {
    return triggerConsolidation(pi, store, target, signal, config.consolidationTimeoutMs, target, config);
  });
  ```
- **triggerConsolidation** 流程（`auto-consolidate.ts:55-94`）：
  1. 拼 prompt
  2. `execChildPrompt()` → spawn 子 `pi -p`
  3. 退码 0 → `{ consolidated: true }`；否则返回错误
- **关键设计**：
  - 只重试 1 次（`_retriesLeft - 1` → 0）
  - 必须 `loadFromDisk()` 重新读
  - 没有 consolidator → fall through 到 reject
  - 子进程 timeout 由 `consolidationTimeoutMs`（默认 60s）控制

### ③ `reject`（直接拒绝）

最简单：直接 `return this.memoryFullError(target, content.length)`，返回：

```ts
{ success: false, error: "Memory is full (X/Y chars). Use a shorter entry, remove old ones, or run /memory-consolidate." }
```

- 不调 LLM
- 不淘汰任何条目
- LLM 收到错误可**自己决定**调 `/memory-consolidate` 合并，或者缩短内容

---

## 与 `/memory-consolidate` 命令的区别

| 触发 | 路径 | Timeout |
|---|---|---|
| **`auto-consolidate` 策略**（被动） | `MemoryStore._add()` 容量溢出 → 子进程 | `consolidationTimeoutMs`（默认 **60s**） |
| **`/memory-consolidate` 命令**（主动） | `registerConsolidateCommand` → 4 个 target（memory/user/failure/project）串行跑 | 手动模式自动 **≥ 180s**（`Math.max(timeoutMs, 180000)`） |

两者用**同一个** `triggerConsolidation()` 函数。

### `/memory-consolidate` 命令的具体行为

```ts
pi.registerCommand("memory-consolidate", {
  description: "Manually trigger memory consolidation to free up space",
  handler: async (_args, ctx) => {
    const manualTimeoutMs = Math.max(timeoutMs, 180000);  // 至少 180s
    const targets = [
      { label: "memory", store, target: "memory", toolTarget: "memory" },
      { label: "user",   store, target: "user",   toolTarget: "user" },
      { label: "failure",store, target: "failure",toolTarget: "failure" },
    ];
    if (projectStore) {
      targets.push({
        label: projectName ? `project:${projectName}` : "project",
        store: projectStore,
        target: "memory",
        toolTarget: "project",
      });
    }

    for (const item of targets) {
      const entries = entriesForTarget(item.store, item.target);
      if (entries.length === 0) { results.push("...empty, skip"); continue; }
      const result = await triggerConsolidation(pi, item.store, item.target, ctx.signal, manualTimeoutMs, item.toolTarget, llmConfig);
      if (result.consolidated) {
        await item.store.loadFromDisk();    // ← 同样必须 reload
        results.push(`${item.label}: ✅ consolidated`);
      } else {
        results.push(`${item.label}: ❌ ${result.error}`);
      }
    }
  },
});
```

**关键点**：
- 串行跑 3~4 个 target（不是并发）
- 每个 target 独立 `triggerConsolidation` + `loadFromDisk`
- 空 target 自动 skip
- 失败不中断后续 target

---

## Consolidation Failure 错误信息

`auto-consolidate.ts:21-35` 格式化错误：

```ts
function describeConsolidationFailure(result, timeoutMs) {
  if (result.killed || result.code === 143) {
    return `Consolidation subprocess was terminated (likely timeout or cancellation). 
            Timeout: ${timeoutMs}ms. Consider increasing consolidationTimeoutMs.`;
  }
  return `Consolidation process exited with code ${result.code}: ${stderr?.slice(0, 200) || "unknown error"}`;
}
```

- `code === 143` = SIGTERM（被 signal 杀掉）
- `killed === true` = 也视为终止
- 都提示用户**调大 `consolidationTimeoutMs`**

---

## 整体流程图

```
memory.add() / addFailure() / addUser() / addProject()
    │
    ▼
MemoryStore._add(target, content, signal, retriesLeft=1)
    │
    ├─ newTotal <= limit
    │   ├─ 写入内存
    │   └─ saveToDisk() 原子写
    │
    └─ newTotal > limit
        │
        ├─ strategy === "fifo-evict"
        │   └─ fifoEvictAndAdd() → shift 头部直到装下 → saveToDisk
        │
        ├─ strategy === "auto-consolidate" && consolidator && retriesLeft > 0
        │   └─ consolidator(target) = triggerConsolidation()
        │       │
        │       └─ execChildPrompt() → pi.exec("pi", ["-p", "--no-session", ..., prompt])
        │           │
        │           ├─ 退码 0
        │           │   ├─ loadFromDisk()         ← CRITICAL
        │           │   └─ _add(..., retriesLeft-1)  ← 重试 1 次
        │           │
        │           ├─ 退码 != 0 且是 override 失败
        │           │   └─ 自动 retry（剥 --model/--thinking）再 exec 一次
        │           │
        │           └─ 退码 != 0（其他错误）
        │               └─ return { consolidated: false, error: "..." }
        │
        └─ strategy === "reject" || retriesLeft === 0 || 没有 consolidator
            └─ memoryFullError()
                └─ return { success: false, error: "Memory is full..." }
```

---

*本主题其他页面：[[INDEX]] · [[概览]] · [[系统提示词注入]] · [[后台机制 - 三大处理器]]*
