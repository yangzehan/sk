---
title: Turn
type: concept
created: 2026-04-06
related:
  - "[[Thread]]"
  - "[[ThreadItem]]"
---

# Turn 对话轮次

Turn 表示对话的一次完整轮次，通常以用户消息开始，以代理消息结束。

## 类型定义

```typescript
type Turn = {
  id: string;
  items: Array<ThreadItem>;  // 轮次中的项目列表
  status: TurnStatus;      // 轮次状态
  error: TurnError | null; // 错误信息（仅在 failed 状态时填充）
};
```

## TurnStatus

```typescript
type TurnStatus = 
  | "pending"    // 等待处理
  | "inProgress" // 进行中
  | "completed"  // 已完成
  | "failed";   // 失败
```

## 生命周期通知

- `TurnStartedNotification` - 轮次开始
- `TurnCompletedNotification` - 轮次完成
- `TurnPlanUpdatedNotification` - 计划更新
- `TurnDiffUpdatedNotification` - 差异更新

## 来源

- Schema: `raw/codex-app-server-schema/ts/v2/Turn.ts`
- 详见: [[openaicodex]]
