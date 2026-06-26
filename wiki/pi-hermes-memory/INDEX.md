---
title: pi-hermes-memory 解析
type: index
created: 2026-06-26
updated: 2026-06-26
tags: [pi, pi-hermes-memory, hermes, memory, persistence, extension, pi-coding-agent]
source:
  - "[[原始资料未建立]]"  # 本主题基于本地安装的源码分析，无外部原始资料
related:
  - "[[Skill Files 角色定义]]"
  - "[[Lesson 2 角色与工具]]"
  - "[[Claude Tag 数据与记忆]]"
  - "[[LLM-Wiki/核心理念]]"
---

# pi-hermes-memory 解析

> **pi-hermes-memory** v0.7.20 是 [[pi-coding-agent]]（Pi 编码 Agent）的第三方扩展（pi-package）。
> 主题：**为 Pi 添加持久化 memory + 会话搜索 + 经验教训**。
> 来源：从 Hermes agent 移植而来，作者 chandra447。

本主题记录对其源码的逐行解析，包含注入机制、配置项、后台处理器、子进程设计、纠错检测、提示词构造等。

---

## 知识分类

### [[概览]]
插件定位、版本、依赖、入口、**9 个命令 / 4 个工具**清单、**28 个配置项**速查

### [[系统提示词注入]]
- `memoryMode` 二选一：`policy-only`（默认）/ `legacy-inject`
- `memoryPolicyStyle` 四选一：`full`（默认）/ `compact` / `custom` / `none`
- `legacy-inject` 模式的 4 段注入规则（全局 MEMORY / USER / failure / 项目 memory）
- Frozen snapshot 设计、`<memory-context>` fence 隔离

### [[后台机制 - 三大处理器]]
- **Background Review**：每 10 turn / 15 tool call 触发，fire-and-forget 子进程
- **Session Flush**：compaction 前 await，shutdown 时 fire-and-forget
- **Correction Detection**：2-pass 文本匹配（强/弱/否定/指令词），速率限制 3 turn
- 三个事件生命周期对比：`turn_end` / `session_before_compact` / `session_shutdown`

### [[auto-consolidate 子进程设计]]
- `pi -p --no-session --no-extensions -e OWN_EXTENSION_PATH` 完全独立子进程
- 三大 overflow 策略：`auto-consolidate`（默认）/ `reject` / `fifo-evict`
- `consolidationTimeoutMs` 兜底 + 失败后自动 retry（剥 override 重试）
- 主进程 `loadFromDisk()` 强制刷新（CRITICAL）

---

## 关键配置速记

| 配置 | 默认 | 改法 |
|---|---|---|
| `memoryMode` | `policy-only` | `~/.pi/agent/hermes-memory-config.json` |
| `memoryPolicyStyle` | `full` | 同上；含 `none` / `compact` / `custom` |
| `memoryDir` | `~/.pi/agent/pi-hermes-memory` | 同上；含 `~` 展开 |
| `nudgeInterval` / `nudgeToolCalls` | 10 / 15 | 关闭后台：`reviewEnabled: false` |
| `failureInjectionEnabled` | `true` | legacy-inject 模式下注入最近 7 天最多 5 条 failure |
| `consolidationTimeoutMs` | 60000 | 手动 `/memory-consolidate` 自动 ≥ 180000 |
| `autoConsolidate` (旧) | `true` | 等同 `memoryOverflowStrategy: "auto-consolidate"` |
| `memoryOverflowStrategy` (新) | `auto-consolidate` | 三选一：`auto-consolidate` / `reject` / `fifo-evict` |

→ **必须重启 pi 才能生效**（`/reload` 不会重读该配置文件）

---

## 源码路径

- 安装包：`/home/yzh/.pi/agent/npm/node_modules/pi-hermes-memory/`
- 数据目录：`/home/yzh/.pi/agent/pi-hermes-memory/`（含 `sessions.db`、项目 `MEMORY.md` / `USER.md` / `failure.md`）
- 入口：`src/index.ts`（通过 `package.json` 的 `"pi": { "extensions": ["./src/index.ts"] }` 注册）
- 关键文件：
  - `src/system-prompt.ts`（不在本扩展内）→ 实际是 `core/system-prompt.js` in pi-coding-agent
  - `src/prompt-context.ts` —— 注入逻辑入口
  - `src/store/memory-store.ts` —— MemoryStore CRUD + 注入渲染
  - `src/handlers/background-review.ts` —— 后台 review
  - `src/handlers/session-flush.ts` —— session flush
  - `src/handlers/correction-detector.ts` —— 纠正检测
  - `src/handlers/auto-consolidate.ts` —— consolidation
  - `src/handlers/pi-child-process.ts` —— 子进程 `pi -p` 调用
  - `src/config.ts` —— 28 个配置项 + loadConfig
  - `src/constants.ts` —— 所有 prompt 字符串 + 正则模式

---

## 关联概念

- [[Skill Files 角色定义]] —— pi-hermes-memory 的 `skill_manage` 工具实现了一套 SKILL.md 三层渐进式披露
- [[Lesson 2 角色与工具]] —— 持久化 memory 是 Pi 的"长期记忆"角色
- [[Claude Tag 数据与记忆]] —— Claude Tag 的记忆作用域（Org/WS/Private） vs pi-hermes-memory 的 4 个 target（user/memory/project/failure）
- [[LLM-Wiki/核心理念]] —— 整个知识库方法论
- [[实践/INDEX|实践]] —— 实战案例可放这里

---

## 全文搜索关键词

下次遇到相关问题，可直接用 obsidian-cli 搜索：

```bash
obsidian search query="pi-hermes-memory"     # 全部
obsidian search query="memoryMode"
obsidian search query="legacy-inject"
obsidian search query="auto-consolidate"
obsidian search query="correction"
obsidian search query="background review"
obsidian search query="session flush"
obsidian search query="持久化记忆"
```

或用 vault 内的 `Cmd+O` 快速跳转 → `pi-hermes-memory` 相关页面。

---

*建立于 2026-06-26。基于本地安装的 pi-hermes-memory@0.7.20 源码。*
