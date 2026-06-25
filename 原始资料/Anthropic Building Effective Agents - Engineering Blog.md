---
title: "Building Effective Agents — Anthropic Engineering"
source: "https://www.anthropic.com/engineering/building-effective-agents"
author: "Erik S. and Barry Zhang (Anthropic)"
published: 2024-12-19 (estimated)
retrieved: 2026-06-25
description: "Anthropic's foundational engineering blog post defining what agents are, distinguishing them from workflows, and presenting common agentic patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, autonomous agents)."
tags:
  - "clippings"
  - "claude"
  - "agent"
  - "engineering"
related-products:
  - "Claude Agent SDK"
  - "Strands Agents SDK"
  - "Rivet"
  - "Vellum"
---

# Building Effective Agents — Anthropic Engineering

## 引言

Over the past year, we've worked with dozens of teams building large language model (LLM) agents across industries. Consistently, the most successful implementations weren't using complex frameworks or specialized libraries. Instead, they were building with **simple, composable patterns**.

## 智能体的官方定义

### 核心定义

> **Agents are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.**

中文翻译：
> **智能体是 LLM 动态指导自身流程和工具使用的系统，在如何完成任务方面保持控制权。**

### 区分：Agentic Systems 中的 Workflows vs Agents

Anthropic 把所有这些变体统称为 **agentic systems**，但在架构上做了重要区分：

| 类型 | 定义 | 控制流 |
|------|------|--------|
| **Workflows（工作流）** | LLM 和工具通过**预定义代码路径**编排 | 代码路径预定义 |
| **Agents（智能体）** | LLM **动态指导**自身流程和工具使用 | LLM 自主决定 |

> **关键差异**：智能体不是按固定脚本运行，而是自主决定如何达成用户目标。

## 何时（不）使用智能体

### 推荐原则

> **找到最简单的方案，只有在需要时才增加复杂性。**

- 不一定需要构建 agentic 系统
- Agentic systems 通常以**延迟和成本**换取更好的任务性能
- 许多应用中，**优化单次 LLM 调用 + RAG + 上下文示例**就够了

### Workflows vs Agents 选择

- **Workflows**：任务定义明确时，可预测性和一致性高
- **Agents**：灵活性和模型驱动决策是规模化所需时

## 构建基块：增强型 LLM (Augmented LLM)

智能体系统的基础构建块：

- **检索**（RAG）
- **工具**（Tools）
- **记忆**（Memory）

当前模型可以**主动**使用这些能力——生成自己的搜索查询、选择合适的工具、决定保留什么信息。

## 五大 Workflow 模式

> 这些是 workflows，不是 agents，但常作为智能体的组件。

### 1. Prompt Chaining（提示链）
- 将任务分解为步骤序列
- 每个 LLM 调用处理上一步输出
- 可在中间步骤加**门控检查（gate）**
- **用例**：先生成营销文案，再翻译；先写大纲检查，再写正文

### 2. Routing（路由）
- 对输入**分类**并导向专门的下游任务
- **用例**：客服问题分类（一般咨询、退款、技术支持）走不同流程

### 3. Parallelization（并行化）
- LLM 同时处理任务，程序化汇总输出
- 两种变体：
  - **Sectioning（分割）**：拆分子任务并行
  - **Voting（投票）**：多次执行同一任务获得多样输出
- **用例**：guardrails、内容审核、SWE-bench 多角度评估

### 4. Orchestrator-Workers（编排者-工作者）
- 中央 LLM **动态分解**任务、委派给 worker LLM、综合结果
- **与并行化的关键差异**：子任务**不是预定义**的，由编排者根据具体输入决定
- **用例**：复杂的多文件代码修改、跨源信息搜集

### 5. Evaluator-Optimizer（评估者-优化者）
- 一个 LLM 生成响应，另一个提供评估和反馈，形成循环
- **适用信号**：
  - 人类反馈能显著改进 LLM 响应
  - LLM 能提供有用的反馈
- **用例**：文学翻译、多轮搜索分析

## 真正的 Agents

### 定义

> **Agents** are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.

### 工作方式

1. 从人类用户的**命令或交互式讨论**开始
2. 任务明确后，**自主规划和操作**
3. 执行过程中，从环境获取"ground truth"（工具调用结果、代码执行输出）
4. 可在**检查点暂停**获取人类反馈，或在遇到阻塞时暂停
5. 完成时终止，或到达停止条件（如最大迭代次数）

### 本质

> **Agents can handle sophisticated tasks, but their implementation is often straightforward. They are typically just LLMs using tools based on environmental feedback in a loop.**

智能体通常只是**在循环中根据环境反馈使用工具的 LLM**。

### 何时使用

- **开放性问题**：难以预测所需步骤数
- **无法硬编码固定路径**
- LLM 可能运行**多轮**
- 需要对其决策有**一定信任**
- 适合在**可信任环境中规模化**

> ⚠️ 智能体的自主性意味着**更高成本**和**复合错误**风险。推荐在沙盒环境**广泛测试**并设置适当护栏。

### Anthropic 自己的用例

- **SWE-bench coding agent**：基于任务描述编辑多个文件解决问题
- **Computer use reference implementation**：Claude 使用计算机完成任务

## 三大核心原则

实现智能体时 Anthropic 遵循的三大原则：

1. **Maintain simplicity** — 保持智能体设计的简洁性
2. **Prioritize transparency** — 优先透明性，明确展示智能体的规划步骤
3. **Carefully craft your agent-computer interface (ACI)** — 通过彻底的工具**文档和测试**精心设计智能体-计算机接口

> 类比：ACI 应投入与 HCI（人机交互界面）同等的精力。

## ACI 设计建议

- **Put yourself in the model's shoes**：从模型视角判断工具是否易用
- **Write great docstrings**：像为初级开发者写文档一样写工具描述
- **Test with workbench**：在控制台运行大量示例，观察错误并迭代
- **Poka-yoke your tools**：通过参数设计让模型更难犯错

> 经验数据：在构建 SWE-bench agent 时，**优化工具的时间比优化整体 prompt 更多**。

## 客户案例

### A. 客户支持

- 客服对话 + 工具集成
- 工具：拉取客户数据、订单历史、知识库
- 行动：退款、改工单（程序化处理）
- 成功标准：用户定义的解决方案
- 商业模式：**按成功解决次数**收费（基于使用量）

### B. 编码智能体

- 优势：
  - 代码方案**可通过自动化测试验证**
  - 智能体可基于测试反馈迭代
  - 问题空间定义良好且结构化
  - 输出质量**可客观度量**
- **Anthropic 自己的实现**：能基于 PR 描述解决 SWE-bench Verified 中的真实 GitHub issue

## 总结

成功不在于构建最复杂的系统，而在于**为你的需求构建正确的系统**：

1. 从**简单 prompt** 开始
2. 通过**全面评估**优化
3. 仅在简单方案不足时添加**多步 agentic 系统**

## 链接

- **原文**: <https://www.anthropic.com/engineering/building-effective-agents>
- **Cookbook 示例**: <https://platform.claude.com/cookbook/patterns-agents-basic-workflows>
- **Claude Agent SDK**: <https://platform.claude.com/docs/en/agent-sdk/overview>
- **Anthropic 中的 SWE-bench agent**: <https://www.anthropic.com/research/swe-bench-sonnet>
- **Computer use demo**: <https://github.com/anthropics/anthropic-quickstarts/tree/main/computer-use-demo>