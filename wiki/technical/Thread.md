---
title: Thread
type: concept
created: 2026-04-06
related:
  - "[[Turn]]"
  - "[[ThreadItem]]"
---

# Thread 对话线程

Thread 代表用户与 Codex 代理之间的完整对话会话。

## 类型定义

```typescript
type Thread = {
  id: string;
  preview: string;           // 第一个用户消息
  ephemeral: boolean;        // 是否临时会话
  modelProvider: string;     // 模型提供商
  createdAt: number;        // 创建时间戳
  updatedAt: number;        // 更新时间戳
  status: ThreadStatus;    // 线程状态
  path: string | null;      // 磁盘路径
  cwd: string;              // 工作目录
  cliVersion: string;       // CLI 版本
  source: SessionSource;    // 来源 (CLI, VSCode, etc.)
  agentNickname: string | null;
  agentRole: string | null;
  gitInfo: GitInfo | null;
  name: string | null;      // 线程名称
  turns: Array<Turn>;       // 对话轮次列表
};
```

## 生命周期

| 操作 | 请求 | 响应 |
|------|------|------|
| 启动 | `ThreadStartParams` | `ThreadStartResponse` |
| 恢复 | `ThreadResumeParams` | `ThreadResumeResponse` |
| 读取 | `ThreadReadParams` | `ThreadReadResponse` |
| 归档 | `ThreadArchiveParams` | `ThreadArchiveResponse` |
| Fork | `ThreadForkParams` | `ThreadForkResponse` |
| 回滚 | `ThreadRollbackParams` | `ThreadRollbackResponse` |

## ThreadStatus

```typescript
type ThreadStatus = 
  | "pending"    // 等待启动
  | "running"    // 运行中
  | "completed"  // 已完成
  | "failed"    // 失败
  | "archived";  // 已归档
```

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/Thread.ts`
- 详见: [[openaicodex]]
