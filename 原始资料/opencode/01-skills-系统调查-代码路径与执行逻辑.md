# OpenCode 原始资料 - Skills 系统调查

> 会话来源: opencode 桌面端 /skills 弹窗实施
> 调查时间: 2026-06-04
> 仓库路径: `/home/yzh/job/opencode` (分支 dev)

## 调研目标

> 为什么桌面端不像 TUI 那样提供 `/skills` 命令弹窗，而是把 skill 当作普通斜杠命令内联显示？

## 关键发现（TL;DR）

| 维度 | TUI | 桌面端 |
|---|---|---|
| 自动补全 | **显式排除** `source === "skill"` | 不过滤，全显示 |
| 入口 | 显式注册 `slashName: "skills"` 弹窗 | 无 |
| 数据源 | `sdk.client.app.skills()` | `sync.data.command`（已在 store）|
| 选中后行为 | 把 `/<skill> ` 写回输入框 | - |

## 后端代码路径

### Skill 发现与注册

`packages/opencode/src/skill/index.ts`
- 5 个发现位置：
  1. 内置：`customize-opencode`（硬编码）
  2. 全局目录：`~/.claude/skills/**/SKILL.md`、`~/.agents/skills/**/SKILL.md`
  3. 项目目录：向上查找 `.claude/skills/` 和 `.agents/skills/`
  4. opencode 目录：`{skill,skills}/**/SKILL.md`
  5. 配置路径：`cfg.skills?.paths`、`cfg.skills?.urls`（远程 URL）

### Command 注册（skills 作为命令）

`packages/opencode/src/command/index.ts:29-41` —— Command.Info 数据结构，`source` 字段支持 `"command" | "mcp" | "skill"`

`packages/opencode/src/command/index.ts:141-152` —— 把 skills 注册为 commands：
```typescript
for (const item of yield* skill.all()) {
  if (commands[item.name]) continue
  commands[item.name] = {
    name: item.name,
    description: item.description,
    source: "skill",
    get template() { return item.content },
    hints: [],
  }
}
```

### API 端点

`packages/opencode/src/server/routes/instance/httpapi/groups/instance.ts:159-168`
```typescript
HttpApiEndpoint.get("skill", InstancePaths.skill, {
  query: WorkspaceRoutingQuery,
  success: described(Schema.Array(Skill.Info), "List of skills"),
}).annotateMerge(
  OpenApi.annotations({
    identifier: "app.skills",
    summary: "List skills",
  }),
)
```

前端调用：`sdk.client.app.skills()` 返回 `Skill.Info[]`

## TUI 实现（参考模板）

### 弹窗组件

`packages/opencode/src/cli/cmd/tui/component/dialog-skill.tsx` (37 行)
- 调 `sdk.client.app.skills()` 拿数据
- 用 `DialogSelect` 渲染（带搜索框）
- 选中后回调 `onSelect: (skill: string) => void`

### 命令注册

`packages/opencode/src/cli/cmd/tui/component/prompt/index.tsx:598-617`
```typescript
{
  title: "Skills",
  name: "prompt.skills",
  category: "Prompt",
  slashName: "skills",          // ← 作为斜杠命令
  run: () => {
    dialog.replace(() => (
      <DialogSkill
        onSelect={(skill) => {
          input.setText(`/${skill} `)
          ...
        }}
      />
    ))
  },
}
```

### 自动补全中排除

`packages/opencode/src/cli/cmd/tui/component/prompt/autocomplete.tsx:547-548`
```typescript
for (const serverCommand of sync.data.command) {
  if (serverCommand.source === "skill") continue   // ← 关键
  ...
}
```

### SlashName 提取机制

`packages/opencode/src/cli/cmd/tui/keymap.tsx:253-283` —— `useCommandSlashes` 从 keymap 提取 `slashName` 字段

## 桌面端实现（待改造）

### 斜杠命令处理

`packages/app/src/components/prompt-input.tsx:642-664`
```typescript
const slashCommands = createMemo<SlashCommand[]>(() => {
  const builtin = command.options
    .filter((opt) => !opt.disabled && !opt.id.startsWith("suggested.") && opt.slash)
    .map((opt) => ({...}))

  const custom = sync.data.command.map((cmd) => ({     // ← 不过滤 skill
    id: `custom.${cmd.name}`,
    trigger: cmd.name,
    title: cmd.name,
    description: cmd.description,
    type: "custom" as const,
    source: cmd.source,
  }))

  return [...custom, ...builtin]
})
```

`handleSlashSelect` (line 666-682)：
- `cmd.type === "custom"` → 插入文本到编辑器
- `cmd.type === "builtin"` → `clearEditor() + prompt.set(empty) + command.trigger(id, "slash")`

### SlashCommand 类型

`packages/app/src/components/prompt-input/slash-popover.tsx:10-18`
```typescript
export interface SlashCommand {
  id: string
  trigger: string
  title: string
  description?: string
  keybind?: string
  type: "builtin" | "custom"
  source?: "command" | "mcp" | "skill"   // ← 已有 skill 类型支持
}
```

渲染（line 118-127）显示 source 的徽章（custom / mcp / skill）

### 命令注册

`packages/app/src/pages/session/use-session-commands.tsx` (584 行)
- `withCategory(category)` 高阶工厂
- 每组命令：`sessionCmds()`、`mcpCmds()`、`modelCmds()` ...
- 在 `command.register("session", () => [...])` 中注册
- `mcpCmds` 是最相似的模板：注册 `/mcp` 命令 + 打开 `DialogSelectMcp` 弹窗

### 弹窗模板

`packages/app/src/components/dialog-select-mcp.tsx` (111 行)
- `Dialog` + `List` 组合（不是 DialogSelect）
- `List` 组件支持 `search` + `filterKeys` + `emptyMessage` + `sortBy` + `onSelect` + children render
- 数据从 `sync.data` 或 SDK 拿

### Dialog 系统

`packages/ui/src/context/dialog.tsx`
- `useDialog()` 拿到 dialog context
- `dialog.show(() => <Component />)` 打开弹窗
- `dialog.close()` 关闭
- Escape 自动关闭

### List 组件

`packages/ui/src/components/list.tsx`
- 自动过滤搜索
- 分组显示
- 键盘导航
- 加载/空状态提示

### SDK

`packages/app/src/context/sdk.tsx` 提供 `useSDK()`，底层 `sdk.client.app.skills()` 已实现

## i18n 架构

`packages/app/src/i18n/` —— 18 个文件（en + 17 locale）

- `en.ts` 主文件
- 其他文件：`import { dict as en } from "./en"` + `type Keys = keyof typeof en`（类型强制 key 集合对齐）
- 关键现有 key：
  - `command.category.{session,mcp,model,...}` —— 命令分类
  - `command.{mcp,model,session,...}.{...}` —— 具体命令
  - `dialog.{mcp,lsp,plugins,...}.{title,description,empty}` —— 弹窗
  - `prompt.slash.badge.{custom,skill,mcp}` —— 斜杠徽章
- `parity.test.ts` 验证完整性

## 关键决策点（用户已确认）

1. **过滤范围**：只过滤 skill（不动 mcp）
2. **选中后行为**：写回输入框 `/<skill> ` + 光标在末尾
3. **命令分类**：新建 `command.category.skill`
4. **i18n**：全量更新 18 个语言
