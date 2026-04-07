---
title: MCP Integration
type: concept
created: 2026-04-06
related:
  - "[[ThreadItem]]"
---

# MCP Integration

MCP (Model Context Protocol) 允许 Codex 调用外部工具。

## MCP 工具调用

```typescript
type McpToolCall = {
  id: string;
  server: string;    // MCP 服务器名称
  tool: string;     // 工具名称
  status: McpToolCallStatus;
  arguments: JsonValue;
  result: McpToolCallResult | null;
  error: McpToolCallError | null;
  durationMs: number | null;
};
```

## McpToolCallStatus

```typescript
type McpToolCallStatus = 
  | "pending"     // 等待执行
  | "started"     // 开始执行
  | "inProgress"  // 执行中
  | "completed"   // 已完成
  | "failed";    // 失败
```

## MCP 服务器管理

| 操作 | 请求 | 响应 |
|------|------|------|
| 列表状态 | `ListMcpServerStatusParams` | `ListMcpServerStatusResponse` |

## MCP 认证

```typescript
type McpAuthStatus = 
  | "unauthenticated"
  | "authenticating"
  | "authenticated"
  | "error";
```

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/McpToolCallStatus.ts`
- 详见: [[openaicodex]]
