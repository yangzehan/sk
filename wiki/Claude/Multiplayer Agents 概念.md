---
title: Multiplayer Agents 概念
type: concept
created: 2026-06-25
source: "[[Building Effective Human-Agent Teams - Claude Blog]]"
related:
  - "[[构建有效的人机协作团队]]"
  - "[[Claude Tag]]"
---

# Multiplayer Agents 概念

## 定义

**Multiplayer Agents**（多人协作智能体）是 Anthropic 提出的概念，指能**同时与多个人类协作**的 AI 模型。它与"常规 agent"有相似之处（都有记忆、技能），但有三个本质差异：

## 三大核心能力

### 1. 持久记忆 (Persistent Memory)

> 智能体能**跨会话**记住团队目标，并据此调整执行方向。

- 与人类记忆不同，智能体的记忆完全**由文本构成**
- 必须主动写入文档/对话历史，否则会"遗忘"
- 详见：<https://platform.claude.com/docs/en/managed-agents/memory>

### 2. 独立凭证 (Own Credentials)

> 智能体**不借用人类账号**，拥有独立的身份和权限。

- 优势：可在**安全、可预测的护栏**内运行
- 优势：智能体活动独立审计、追踪
- 详见：<https://www.anthropic.com/engineering/managed-agents>

### 3. 持续广泛的信息访问 (Ongoing Broad Access)

> 智能体**持续学习**组织运作方式，并据此执行任务。

- 类比：新员工第一周读公司文档的过程，但**速度极快**
- 是智能体从"工具"变成"队友"的关键能力
- 详见：<https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>

## 与传统 Agent 的对比

| 特性 | 常规 Agent | Multiplayer Agent |
|------|----------|------------------|
| 用户数 | 单人 | 多人类同时 |
| 记忆 | 会话级 | **持久跨会话** |
| 凭证 | 借用人类 | **独立凭证** |
| 工作位置 | 聊天窗口/IDE | **团队协作工具**（Slack） |
| 信息范围 | 任务相关 | **组织全局** |
| 角色 | 助手 | **团队成员** |

## 技术基础 vs 团队文化

智能体具备上述三大能力是**必要条件**，但**不充分**。

> **真正的成功 = 技术能力 + 团队工作方式与共享规范**

Anthropic 的经验集中在后者——也就是 [[构建有效的人机协作团队|四大经验教训]] 中描述的协作模式。

## 来源

- **原始博客**: [[Building Effective Human-Agent Teams - Claude Blog]]
- **相关产品**: [[Claude Tag]]