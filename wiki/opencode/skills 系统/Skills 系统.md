---
title: Skills 系统
type: system
created: 2026-06-04
updated: 2026-06-04
tags: [opencode, skills, command, discovery, api]
---

# Skills 系统

> 范围: skills 的发现、注册、API、命令暴露全流程

## 概述

Skill 是 OpenCode 的一项功能：让用户通过 SKILL.md 文件定义可复用的提示模板或工作流。Skills 可以从多个位置发现，注册为 commands，最终通过 `/<skill> ` 触发。

## 发现机制

文件: `packages/opencode/src/skill/index.ts`

Skills 从 5 个位置发现：

| 优先级 | 来源 | 路径 |
|---|---|---|
| 1 | 内置 | `customize-opencode`（硬编码） |
| 2 | 用户全局 | `~/.claude/skills/**/SKILL.md` |
| 3 | 用户全局 | `~/.agents/skills/**/SKILL.md` |
| 4 | 项目目录 | 向上查找 `.claude/skills/` 和 `.agents/skills/` |
| 5 | opencode 目录 | `{skill,skills}/**/SKILL.md` |
| 6 | 配置 | `cfg.skills?.paths`（用户配置） |
| 7 | 远程 | `cfg.skills?.urls`（从 URL 拉取） |

**SKILL.md 格式**：
```markdown
---
name: my-skill
description: A test skill for validation
---
```

## 注册为 Command

文件: `packages/opencode/src/command/index.ts:141-152`

Skills 在后端被注册为 commands，共享统一的 command 抽象：

```typescript
export const Info = Schema.Struct({
  name: Schema.String,
  description: Schema.optional(Schema.String),
  agent: Schema.optional(Schema.String),
  model: Schema.optional(Schema.String),
  source: Schema.optional(Schema.Literals(["command", "mcp", "skill"])),  // ← 关键
  template: Schema.Unknown,
  subtask: Schema.optional(Schema.Boolean),
  hints: Schema.Array(Schema.String),
})

// 注册逻辑
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

`source` 字段是关键标识，前端用这个字段区分：
- `"command"` - 自定义命令（来自 `.opencode/command/`）
- `"mcp"` - MCP 工具命令
- `"skill"` - Skill 命令

## API 端点

文件: `packages/opencode/src/server/routes/instance/httpapi/groups/instance.ts:159-168`

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

**前端调用**：
```typescript
const result = await sdk.client.app.skills()
const skills: Skill.Info[] = result.data ?? []
```

**响应结构**：
```typescript
Skill.Info = {
  name: string
  description?: string
  // ... 其他字段
}
```

## 触发流程

### 后端

1. 用户在 prompt 中输入 `/my-skill ...`
2. 后端解析命令，识别 `my-skill` 是 skill（`source === "skill"`）
3. 读取 `item.template`（SKILL.md 的内容）
4. 注入到 LLM 上下文中

### 前端命令处理

详见 [[桌面端命令系统]]

### TUI 弹窗处理

详见 [[桌面端 vs TUI 架构对比]]

## 数据流图

```
SKILL.md 文件
    ↓ (skill.all() 发现)
skill.all(): Skill.Info[]
    ↓ (command/index.ts 注册为 commands)
commands: Map<string, Command.Info>  // source = "skill"
    ↓ (sync 同步)
sync.data.command: Command.Info[]  // 前端 store
    ↓
desktop: slashCommands memo → 列表显示
tui: autocomplete → 列表显示
    ↓
/skills 弹窗 → sdk.client.app.skills() 独立拉取
```

## Source 字段使用情况

| Source | 触发方式 | 斜杠列表 | 弹窗入口 |
|---|---|---|---|
| `"command"` | `/<name> ` 直接发送 | 显示（无徽章） | - |
| `"mcp"` | `/<name> ` 直接发送 | 显示（mcp 徽章） | `/mcp` 浏览/切换 |
| `"skill"` | `/<name> ` 直接发送 | 桌面端：显示（skill 徽章）<br>TUI：不显示 | `/skills` 浏览/选择 |

## SDK 端点

文件: `packages/sdk/js/src/v2/gen/sdk.gen.ts:488-516`

```typescript
public skills<ThrowOnError extends boolean = false>(
  parameters?: {
    directory?: string
    workspace?: string
  },
  options?: Options<never, ThrowOnError>,
) {
  return (options?.client ?? this.client).get<AppSkillsResponses, AppSkillsErrors, ThrowOnError>({
    url: "/skill",
    ...options,
    ...params,
  })
}
```

## 相关文档

- [[桌面端 vs TUI 架构对比]]
- [[桌面端命令系统]]
- [[实施记录 - /skills 弹窗]]
- [原始资料: 01-skills-系统调查-代码路径与执行逻辑](../../原始资料/opencode/01-skills-系统调查-代码路径与执行逻辑.md)
