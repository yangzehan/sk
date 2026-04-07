---
title: ThreadItem
type: concept
created: 2026-04-06
related:
  - "[[Turn]]"
  - "[[Thread]]"
---

# ThreadItem 对话项目

ThreadItem 表示作为 turn 一部分的用户输入和代理输出，是对话的基本组成单元。

## 类型定义

ThreadItem 是一个 discriminated union，包含以下类型：

### 用户消息
```typescript
{ "type": "userMessage", id: string, content: Array<UserInput> }
```

### Agent 消息
```typescript
{ "type": "agentMessage", id: string, text: string, phase: MessagePhase | null }
```

### 推理过程
```typescript
{ "type": "reasoning", id: string, summary: Array<string>, content: Array<string> }
```

### 计划
```typescript
{ "type": "plan", id: string, text: string }
```

### 命令执行
```typescript
{ "type": "commandExecution", id: string,
  command: string;
  cwd: string;
  processId: string | null;
  source: CommandExecutionSource;
  status: CommandExecutionStatus;
  commandActions: Array<CommandAction>;
  aggregatedOutput: string | null;
  exitCode: number | null;
  durationMs: number | null;
}
```

### 文件更改
```typescript
{ "type": "fileChange", id: string, changes: Array<FileUpdateChange>, status: PatchApplyStatus }
```

### MCP 工具调用
```typescript
{ "type": "mcpToolCall", id: string, server: string, tool: string, status: McpToolCallStatus, ... }
```

### 动态工具调用
```typescript
{ "type": "dynamicToolCall", id: string, tool: string, status: DynamicToolCallStatus, ... }
```

### Web 搜索
```typescript
{ "type": "webSearch", id: string, query: string, action: WebSearchAction | null }
```

### 图片查看
```typescript
{ "type": "imageView", id: string, path: string }
```

### 图片生成
```typescript
{ "type": "imageGeneration", id: string, status: string, revisedPrompt: string | null, result: string }
```

### 审查模式
```typescript
{ "type": "enteredReviewMode", id: string, review: string }
{ "type": "exitedReviewMode", id: string, review: string }
```

### 上下文压缩
```typescript
{ "type": "contextCompaction", id: string }
```

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/ThreadItem.ts`
- 详见: [[openaicodex]]
