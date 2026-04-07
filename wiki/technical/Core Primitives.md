---
title: Core Primitives
type: concept
created: 2026-04-06
related:
  - "[[Protocol]]"
  - "[[Lifecycle]]"
---

# Core Primitives 核心原语

API 暴露三个顶层原语，表示用户与 Codex 之间的交互：

## Thread (对话线程)

一个用户与 Codex 代理之间的对话，包含多个 turn。

## Turn (对话轮次)

对话的一次轮次，通常以用户消息开始，以代理消息结束。每个 turn 包含多个 item。

## Item (项目)

表示作为 turn 一部分的用户输入和代理输出，作为持久化内容用于未来对话的上下文。

项目类型包括：
- `userMessage` - 用户消息
- `agentMessage` - 代理消息
- `reasoning` - 推理过程
- `commandExecution` - 命令执行
- `fileChange` - 文件更改
- `mcpToolCall` - MCP 工具调用

## 来源

- 详见: [[Thread]]、[[Turn]]、[[ThreadItem]]
- 原始文档: [[openaicodex]]
