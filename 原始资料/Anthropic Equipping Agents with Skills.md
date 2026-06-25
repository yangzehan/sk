---
title: "Equipping Agents for the Real World with Agent Skills — Anthropic Engineering"
source: "https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills"
author: "Barry Zhang, Keith Lazuka, Mahesh Murag (Anthropic)"
published: 2025 (initial) / Updated 2025-12-18
retrieved: 2026-06-25
description: "Anthropic's official engineering post introducing Agent Skills — organized folders of instructions, scripts, and resources that agents can discover and load dynamically to specialize their behavior. Defines the SKILL.md format, progressive disclosure principle, and best practices."
tags:
  - "clippings"
  - "claude"
  - "agent"
  - "skills"
related-products:
  - "Agent Skills"
  - "Claude Code"
  - "Claude Agent SDK"
  - "Claude Developer Platform"
---

# Equipping Agents for the Real World with Agent Skills

## 更新

> **Update**: We've published **Agent Skills** as an open standard for cross-platform portability. (December 18, 2025) — <https://agentskills.io/>

## 引言

As model capabilities improve, we can now build **general-purpose agents** that interact with full-fledged computing environments. Claude Code, for example, can accomplish complex tasks across domains using local code execution and filesystems. But as these agents become more powerful, we need more composable, scalable, and portable ways to equip them with **domain-specific expertise**.

This led us to create **Agent Skills**:

> **Organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks.**

### Skill 的本质

> **Skills extend Claude's capabilities by packaging your expertise into composable resources for Claude, transforming general-purpose agents into specialized agents that fit your needs.**

类比：

> **Building a skill for an agent is like putting together an onboarding guide for a new hire.**

## Skill 的结构（Anatomy of a Skill）

### 最简结构

> **At its simplest, a skill is a directory that contains a `SKILL.md file`.**

`SKILL.md` 必须以 **YAML frontmatter** 开头，包含**必需元数据**：

```yaml
---
name: <skill-name>
description: <skill-description>
---
```

### 三层渐进式披露（Progressive Disclosure）

这是 Agent Skills 的**核心设计原则**：

| 层级 | 内容 | 何时加载 |
|------|------|---------|
| **Level 1** | `name` + `description`（元数据） | 启动时**预加载**到 system prompt |
| **Level 2** | `SKILL.md` 主体内容 | Claude 认为相关时**完整读入**上下文 |
| **Level 3+** | 链接的额外文件（`reference.md`、`forms.md` 等） | Claude 按需**导航发现** |

> **类比**：像一本组织良好的手册——先有目录，再有具体章节，最后有详细附录。Skills 让 Claude **按需加载信息**。

### 为什么渐进式披露重要

> **Agents with a filesystem and code execution tools don't need to read the entirety of a skill into their context window when working on a particular task. This means that the amount of context that can be bundled into a skill is effectively unbounded.**

智能体不需要把整个 skill 读入上下文窗口 → **单个 skill 可包含的有效上下文是无限的**。

## Skill 的工作流程（Context Window 中的触发）

以 PDF skill 为例：

1. **初始状态**：上下文窗口包含 system prompt + 每个已安装 skill 的元数据 + 用户消息
2. **触发**：Claude 调用 Bash 工具读取 `pdf/SKILL.md`
3. **深入**：Claude 选择读取 skill 附带的 `forms.md` 文件
4. **执行**：Claude 现在使用 PDF skill 的相关指令完成用户任务

## Skills 与代码执行

Skills 可以包含**供 Claude 作为工具执行的代码**：

> **Large language models excel at many tasks, but certain operations are better suited for traditional code execution.**

- **效率**：通过 token 生成排序列表远比运行排序算法昂贵
- **可靠性**：代码提供传统软件才有的**确定性可靠性**

### 例子

PDF skill 包含一个**预写好的 Python 脚本**，用于读取 PDF 并提取所有表单字段：
- Claude 可运行此脚本
- **无需**将脚本或 PDF 加载到上下文中
- 工作流**一致且可重复**

## 开发与评估 Skills 的最佳实践

### 1️⃣ 从评估开始 (Start with evaluation)
- 在代表性任务上运行智能体
- 观察它们**在哪里遇到困难或需要额外上下文**
- **增量**构建 skill 解决这些短板

### 2️⃣ 为规模化而设计 (Structure for scale)
- 当 `SKILL.md` 变得难以管理时，**拆分内容到独立文件**并引用它们
- 互斥或很少一起使用的上下文 → **保持路径分离**以减少 token 使用
- 代码可作为**可执行工具**和**文档**——明确 Claude 应**直接运行脚本**还是**作为参考读入**

### 3️⃣ 从 Claude 的视角思考 (Think from Claude's perspective)
- 监控 Claude 在真实场景中**如何使用 skill**
- 观察意外路径或对某些上下文的**过度依赖**
- 特别关注 `name` 和 `description`——Claude 用它们**决定是否触发 skill**

### 4️⃣ 与 Claude 一起迭代 (Iterate with Claude)
- 在任务过程中，要求 Claude **捕获成功方法和常见错误**到 skill 中
- 如果 Claude 使用 skill 时偏离轨道，要求它**自我反思**哪里出了问题
- 这个过程帮助你**发现 Claude 真正需要什么上下文**，而不是预先猜测

## 安全考虑

> **Skills provide Claude with new capabilities through instructions and code. While this makes them powerful, it also means that malicious skills may introduce vulnerabilities...**

### 建议

- ⚠️ **仅从可信来源**安装 skill
- ⚠️ 安装来自不太可信来源的 skill 时，**使用前彻底审计**
- 仔细阅读 skill 中捆绑的**所有文件**
- 特别关注：
  - 代码依赖
  - 捆绑资源（图片、脚本）
  - 指示 Claude 连接**潜在不可信外部网络源**的指令或代码

## 当前支持范围

Agent Skills 当前支持的平台：

- ✅ Claude.ai
- ✅ Claude Code
- ✅ Claude Agent SDK
- ✅ Claude Developer Platform

## 未来方向

- 持续添加支持 **skill 创建、编辑、发现、共享和使用**全生命周期的功能
- 探索 Skills 如何与 **Model Context Protocol (MCP)** 互补
- **长期愿景**：希望智能体能**自主创建、编辑和评估自己的 Skills**，将自身的行为模式编码为可重用能力

## 链接

- **原文**: <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
- **Agent Skills 开放标准**: <https://agentskills.io/>
- **Anthropic Skills GitHub 仓库**: <https://github.com/anthropics/skills>
- **官方文档**: <https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview>
- **Cookbook**: <https://github.com/anthropics/claude-cookbooks/tree/main/skills>