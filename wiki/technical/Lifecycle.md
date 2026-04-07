---
title: Lifecycle Overview
type: concept
created: 2026-04-06
related:
  - "[[Core Primitives]]"
  - "[[Events]]"
---

# Lifecycle Overview 生命周期概述

## 连接初始化

1. 打开传输连接后，立即发送 `initialize` 请求
2. 发送 `initialized` 通知确认完成
3. 握手完成前，任何其他请求都会被拒绝

## 会话管理

- `thread/start` - 创建新对话
- `thread/resume` - 恢复现有对话
- `thread/fork` - 分支现有对话

## Turn 处理

- `turn/start` - 发送用户输入并开始 Codex 生成
- `turn/steer` - 向正在进行的 turn 添加输入
- `turn/interrupt` - 请求取消正在进行的 turn

## 事件流

连接后持续读取 JSON-RPC 通知：
- `item/started` - 项目开始
- `item/completed` - 项目完成
- `item/agentMessage/delta` - 代理消息增量
- `turn/completed` - turn 完成

## 来源

- 详见原始文档: [[openaicodex]]
