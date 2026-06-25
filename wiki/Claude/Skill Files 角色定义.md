---
title: Skill Files 角色定义（Agent Skills）
type: pattern
created: 2026-06-25
updated: 2026-06-25
source:
  - "[[Anthropic Equipping Agents with Skills]]"
  - "[[Building Effective Human-Agent Teams - Claude Blog]]"
related:
  - "[[Lesson 2 角色与工具]]"
  - "[[智能体官方定义]]"
  - "[[构建有效的人机协作团队]]"
---

# Skill Files 角色定义（Agent Skills）

> **重要提示**：本页面基于两个来源综合而成——
> 1. **Anthropic 官方工程博客**（主要来源，提供完整定义和结构）
> 2. **"Building Effective Human-Agent Teams" 博客**（一句话提及）

---

## 📌 来源一：Anthropic 官方定义（**权威**）

来源：[[Anthropic Equipping Agents with Skills]] · <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>

### 官方定义

> **Agent Skills**: Organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks.

> 中文：**Agent Skills 是组织化的指令、脚本和资源文件夹，智能体可动态发现并加载，以在特定任务上表现更好。**

### 核心作用

> **Skills extend Claude's capabilities by packaging your expertise into composable resources for Claude, transforming general-purpose agents into specialized agents that fit your needs.**

> Skills 通过将专业知识打包为可组合资源来**扩展 Claude 的能力**，将通用智能体转变为**满足需求的专门化智能体**。

### 类比

> **Building a skill for an agent is like putting together an onboarding guide for a new hire.**

> 为智能体构建 skill **就像为新员工准备入职指南**。

---

## Skill 的结构（Anatomy）

### 最简形式

> **At its simplest, a skill is a directory that contains a `SKILL.md` file.**

Skill **是一个目录**，包含一个 `SKILL.md` 文件。

### `SKILL.md` 文件格式

**必须**以 YAML frontmatter 开头：

```yaml
---
name: <skill-name>          # 必需
description: <skill-desc>   # 必需
---
```

> **关键**：`name` 和 `description` 是**仅有的必需字段**。Claude 用这两个字段**决定何时触发该 skill**。

---

## 三层渐进式披露（Progressive Disclosure）

这是 Skill 的**核心设计原则**：

| 层级 | 内容 | 何时加载 |
|------|------|---------|
| **Level 1** | `name` + `description`（元数据） | 启动时**预加载**到 system prompt |
| **Level 2** | `SKILL.md` 主体内容 | Claude 认为相关时**完整读入**上下文 |
| **Level 3+** | 链接的额外文件（`reference.md`、`forms.md` 等） | Claude 按需**导航发现** |

> **类比**：像一本组织良好的手册——先有目录，再有具体章节，最后有详细附录。

### 为什么重要

> **The amount of context that can be bundled into a skill is effectively unbounded.**

智能体不需要把整个 skill 读入上下文 → 单个 skill 可包含的有效上下文**实际上是无限的**。

---

## Skill 工作流程示例（PDF Skill）

以 Anthropic 的 PDF skill 为例：

1. **初始状态**：上下文窗口包含 system prompt + 每个已安装 skill 的元数据 + 用户消息
2. **触发**：Claude 调用 Bash 工具读取 `pdf/SKILL.md`
3. **深入**：Claude 选择读取 skill 附带的 `forms.md` 文件
4. **执行**：Claude 现在使用 PDF skill 的相关指令完成任务

---

## Skill 中的代码执行

Skills 可以包含**供 Claude 作为工具执行的代码**：

- **效率**：通过 token 生成排序列表远比运行排序算法昂贵
- **可靠性**：代码提供**确定性可靠性**

### PDF Skill 例子

PDF skill 包含预写好的 Python 脚本读取 PDF 并提取表单字段：
- Claude 可运行此脚本
- **无需**将脚本或 PDF 加载到上下文
- 工作流**一致且可重复**

---

## 官方推荐的最佳实践

### 1️⃣ 从评估开始（Start with evaluation）
- 在代表性任务上运行智能体
- 观察**在哪里遇到困难或需要额外上下文**
- **增量**构建 skill 解决短板

### 2️⃣ 为规模化而设计（Structure for scale）
- `SKILL.md` 难以管理时**拆分到独立文件**
- 互斥或很少一起使用的上下文 → **保持路径分离**减少 token
- 代码可作为**可执行工具**和**文档**——明确用法

### 3️⃣ 从 Claude 的视角思考（Think from Claude's perspective）
- 监控 Claude **如何使用 skill**
- 观察意外路径或**过度依赖**
- 特别关注 `name` 和 `description`

### 4️⃣ 与 Claude 一起迭代（Iterate with Claude）
- 要求 Claude **捕获成功方法**到 skill 中
- 让 Claude **自我反思**偏离轨道的原因
- 这帮助你**发现真正需要的上下文**，而不是预先猜测

---

## 安全考虑

> **Skills provide Claude with new capabilities through instructions and code. While this makes them powerful, it also means that malicious skills may introduce vulnerabilities...**

### 建议

- ⚠️ **仅从可信来源**安装 skill
- ⚠️ 安装前**彻底审计** skill 内容
- 特别关注：
  - 代码依赖
  - 捆绑资源（图片、脚本）
  - 指示 Claude 连接**不可信外部网络源**的指令或代码

---

## 📌 来源二：Human-Agent Teams 博客（**一句话提及**）

来源：[[Building Effective Human-Agent Teams - Claude Blog]]

原博客关于 skill files **仅有这一处明确说明**（在 Lesson 2 中作为 bullet point）：

> **"Writing skill files to define specific agents' roles helps to make specialization easy, and allows people across the company to quickly stand up other agents of the same type."**

> 编写 skill files 来定义智能体的角色，让专业化变简单，并允许公司内的人快速部署同款智能体。

**仅此而已**——原博客**没有**提供任何 skill file 示例或具体内容模板。

---

## 两个来源的关系

| 维度 | Human-Agent Teams 博客 | Equipping Agents 工程博客 |
|------|---------------------|------------------------|
| **核心关注** | 团队组织方式 | 技术实现细节 |
| **Skill 用途** | 定义智能体角色、方便复制 | 扩展智能体能力、专业化 |
| **Skill 结构** | 未提及 | **完整定义**（SKILL.md 格式） |
| **Skill 内容** | 未提及 | 指令、脚本、资源 |
| **使用方式** | 未提及 | 三层渐进式披露 |

> **结论**：Human-Agent Teams 博客把 skill files 作为**协作工具**提及；Equipping Agents 博客提供**完整技术规范**。两者互补。

---

## 当前支持平台

Agent Skills 当前支持：

- ✅ Claude.ai
- ✅ Claude Code
- ✅ Claude Agent SDK
- ✅ Claude Developer Platform

## 链接

- **官方工程博客**: <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
- **Agent Skills 开放标准**: <https://agentskills.io/>
- **Anthropic Skills GitHub 仓库**: <https://github.com/anthropics/skills>
- **官方文档**: <https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview>
- **Cookbook**: <https://github.com/anthropics/claude-cookbooks/tree/main/skills>

## 来源

- [[Anthropic Equipping Agents with Skills]]
- [[Building Effective Human-Agent Teams - Claude Blog]]