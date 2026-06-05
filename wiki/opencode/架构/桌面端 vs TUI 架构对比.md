---
title: 桌面端 vs TUI 架构对比
type: architecture
created: 2026-06-04
updated: 2026-06-04
tags: [opencode, architecture, desktop, tui, comparison]
---

# 桌面端 vs TUI 架构对比

> 范围: 仅围绕 skills 命令的暴露策略、命令系统、斜杠处理三方面

## 一图看差异

```
TUI 架构（keymap + slashName）              桌面端架构（command.register）
─────────────────────────────              ────────────────────────────────
Keymap                                       CommandProvider
  ↓                                            ↓
getCommandEntries({namespace: "palette"})    command.register(key, callback)
  ↓                                            ↓
useCommandSlashes() → 提取 slashName         CommandOption[]（含 slash 属性）
  ↓                                            ↓
DialogSkill（专门弹窗）                       PromptPopover（统一斜杠弹窗）
  ↑                                            ↑
  排除 skills 在常规命令中                     不过滤，skills 带徽章显示
```

## 核心差异表

| 维度 | TUI | 桌面端 |
|---|---|---|
| 命令系统入口 | `keymap.tsx` + `slashName` 字段 | `command.register` + `CommandOption.slash` |
| 自动补全来源 | `useCommandSlashes()` + `sync.data.command` | `command.options` + `sync.data.command` |
| 弹窗组件 | `@tui/ui/dialog-select`（带搜索） | `Dialog` + `List` 组合（`@opencode-ai/ui`） |
| 选中后行为 | `input.setText(\`/${skill} \`)` | `prompt.set` + DOM cursor 调整 |
| Skill 处理 | 排除 + 单独弹窗 | 内联 + 徽章 |
| Screen 适配 | 终端窄 → 需专门浏览 | 富 UI → 列表可读 |

## 数据流对比

### TUI：`/skills` 触发流程

1. 用户输入 `/sk`
2. `useCommandSlashes` 返回所有 `slashName` 字段
3. 命中 `slashName: "skills"` → 触发 `prompt.skills` 命令
4. `dialog.replace(() => <DialogSkill />)`
5. `DialogSkill` 调 `sdk.client.app.skills()`
6. `DialogSelect` 渲染带搜索的列表
7. 选中 → `input.setText(\`/${skill} \`)` + 清弹窗

### 桌面端：技能显示流程（改动前）

1. 用户输入 `/`
2. `slashCommands` memo 计算（line 642-664）
3. `custom` 部分遍历 `sync.data.command`，**不**过滤 skill
4. 每个 skill 渲染为 `/{name}` + "skill" 徽章
5. 选中 → 插入 `/<skill> ` 到编辑器

### 桌面端：技能显示流程（改动后）

1. 用户输入 `/`
2. `slashCommands` memo 计算
3. `custom` 部分 `filter((cmd) => cmd.source !== "skill")` 过滤
4. Skills 不再出现在主列表
5. 输入 `/sk` → 命中 `slashName: "skills"` builtin 命令
6. `command.trigger("skills.show", "slash")` → `chooseSkill()`
7. 动态 import + `dialog.show(() => <DialogSelectSkill />)`
8. `DialogSelectSkill` 调 `sdk.client.app.skills()`，渲染可搜索列表
9. 选中 → `dialog.close()` + `prompt.set` + DOM focus/cursor

## 关键代码位置

| 模块 | TUI 位置 | 桌面端位置 |
|---|---|---|
| 弹窗组件 | `packages/opencode/src/cli/cmd/tui/component/dialog-skill.tsx` | `packages/app/src/components/dialog-select-skill.tsx` (新建) |
| 命令注册 | `packages/opencode/src/cli/cmd/tui/component/prompt/index.tsx:598-617` | `packages/app/src/pages/session/use-session-commands.tsx` |
| 斜杠提取 | `packages/opencode/src/cli/cmd/tui/keymap.tsx:253-283` | `packages/app/src/components/prompt-input.tsx:642-664` |
| 排除逻辑 | `packages/opencode/src/cli/cmd/tui/component/prompt/autocomplete.tsx:547-548` | `packages/app/src/components/prompt-input.tsx:654` (filter) |
| 渲染 | TUI autocomplete 组件 | `packages/app/src/components/prompt-input/slash-popover.tsx` |

## 设计理念差异

### TUI 的考虑

- 终端屏幕空间有限，技能数量多时列表过长
- 单独弹窗 + 搜索框是更好的浏览方式
- "Skill 不应污染主斜杠列表" —— 显式排除

### 桌面端的考虑

- 富 UI 空间充足，列表展示可控
- 技能通常不多，内联展示已够
- "Skill" 徽章提供视觉区分

### 实践中的痛点

**桌面端的问题**：当用户在 `.claude/skills/` 或自定义 `cfg.skills.paths` 配置了**大量** skill 时（例如 20+），斜杠命令列表会被淹没，找 `/clear`、`/undo` 这些系统命令会变难。

**这次改动后**：对齐 TUI，恢复清晰的主斜杠列表 + 单独 `/skills` 入口。

## 相关文档

- [[skills-系统]] - Skills 的发现、注册、API 详情
- [[实施记录 - /skills 弹窗]] - 本次具体改动
- [原始资料: 01-skills-系统调查-代码路径与执行逻辑](../原始资料/opencode/01-skills-系统调查-代码路径与执行逻辑.md)
- [原始资料: 02-实施计划与设计决策](../原始资料/opencode/02-实施计划与设计决策.md)
