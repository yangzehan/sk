---
title: Codex CLI
type: concept
created: 2026-04-06
related:
  - "[[openaicodex]]"
  - "[[Codex Schema Index]]"
---

# Codex CLI 命令行

Codex CLI (v0.118.0) 提供多种命令模式。

## 交互模式

直接运行 `codex` 启动交互式 TUI：

```
codex [OPTIONS] [PROMPT]
```

## 非交互模式

```
codex exec [OPTIONS] [PROMPT]
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `-c, --config <key=value>` | 覆盖配置值 |
| `-m, --model <MODEL>` | 指定模型 |
| `-s, --sandbox <MODE>` | 沙箱模式 |
| `-C, --cd <DIR>` | 指定工作目录 |
| `--full-auto` | 自动执行模式 |
| `--dangerously-bypass-approvals-and-sandbox` | 危险：绕过所有确认 |

### Sandbox 模式

```
-s, --sandbox <SANDBOX_MODE>
  [possible values: read-only, workspace-write, danger-full-access]
```

### 审批策略

```
-a, --ask-for-approval <APPROVAL_POLICY>
  - untrusted:    只运行可信命令
  - on-request:   模型决定何时请求审批
  - never:       从不请求审批
```

## 子命令

### exec - 非交互执行

运行 Codex 非交互式执行任务。

### review - 代码审查

```
codex review [OPTIONS]
```

### app-server - 应用服务器

启动 Codex 作为 JSON-RPC 2.0 应用服务器：

```
codex app-server [OPTIONS] [COMMAND]

Commands:
  generate-ts           生成 TypeScript 绑定
  generate-json-schema  生成 JSON Schema
```

**常用选项：**

```
--listen <URL>          传输端点 (默认 stdio://)
--ws-auth <MODE>        WebSocket 认证模式
```

### mcp - MCP 服务器管理

```
codex mcp <COMMAND>

Commands:
  list     列出 MCP 服务器
  get      获取 MCP 服务器配置
  add      添加 MCP 服务器
  remove   移除 MCP 服务器
  login    登录 MCP 服务器
  logout   登出 MCP 服务器
```

### login/logout - 账户管理

管理认证凭据。

### sandbox - 沙箱运行

在 Codex 提供的沙箱中运行命令。

### debug - 调试工具

调试功能集合。

### apply - 应用补丁

应用 Codex 生成的最新 diff。

### resume/fork - 会话管理

恢复或 fork 之前的交互会话。

### completion - 补全脚本

生成 shell 补全脚本。

### cloud - 云端任务

浏览 Codex Cloud 任务并在本地应用更改。

### features - 特性开关

检查特性标志。

## 来源

- 原始帮助: [[raw/codex-cli-help.txt]]
- 官方文档: [[openaicodex]]
