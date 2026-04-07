---
title: Command Execution
type: concept
created: 2026-04-06
related:
  - "[[ThreadItem]]"
---

# Command Execution 命令执行

命令执行是 Codex 执行 shell 命令的能力。

## 请求/响应

```typescript
type CommandExecParams = {
  threadId: string;
  command: string;
  cwd?: string;
  timeout?: number;
};

type CommandExecResponse = {
  output: string;
  exitCode: number | null;
};
```

## 命令执行状态

```typescript
type CommandExecutionStatus = 
  | "pending"     // 等待执行
  | "started"     // 开始执行
  | "inProgress"  // 执行中
  | "completed"   // 已完成
  | "failed";    // 失败
```

## 命令操作类型

```typescript
type CommandAction = 
  | { type: "read", path: string }
  | { type: "write", path: string, content: string }
  | { type: "delete", path: string }
  | { type: "createDirectory", path: string }
  | { type: "copy", source: string, destination: string }
  | { type: "move", source: string, destination: string }
  | { type: "changePermission", path: string, mode: string }
  | { type: "networkAccess", host: string, port: number };
```

## 通知

- `CommandExecutionOutputDeltaNotification` - 输出增量通知
- `CommandExecOutputStream` - 输出流类型

## 权限审批

执行命令前可能需要用户审批：
- `CommandExecutionRequestApprovalParams` - 命令执行审批请求
- `CommandExecutionApprovalDecision` - 审批决定

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/CommandExecParams.ts`
- 详见: [[openaicodex]]
