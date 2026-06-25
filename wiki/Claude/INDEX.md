---
title: Claude Knowledge Wiki
type: index
created: 2026-06-25
updated: 2026-06-25
source:
  - "[[Building Effective Human-Agent Teams - Claude Blog]]"
  - "[[Introducing Claude Tag]]"
  - "[[What is Claude Tag]]"
  - "[[Agent identity a new access model for autonomous, team-wide AI]]"
  - "[[Harness design for long-running application development]]"
  - "[[Anthropic Building Effective Agents - Engineering Blog]]"
  - "[[Anthropic Equipping Agents with Skills]]"
---

# Claude Knowledge Wiki

Anthropic 关于人机协作（Human-Agent Collaboration）、Multiplayer Agents、Claude Tag、Agent Skills 与 Harness 设计的知识库。

## 核心概念

- [[构建有效的人机协作团队]] - 四大经验教训概览
- [[Multiplayer Agents 概念]] - 多人协作智能体的定义与三大能力
- [[智能体官方定义]] - Anthropic 对 Agent 的官方定义（Workflows vs Agents 区分）
- [[Agent Identity 访问模型]] - Claude Tag 的底层访问模型：独立凭证、按渠道权限
- [[Claude Tag]] - Anthropic 的多人协作智能体产品

## Claude Tag 操作指南

- [[Claude Tag 设置与管理]] - 管理员配置：三层访问模型、Member Access、Spend Limits
- [[Claude Tag 数据与记忆]] - 数据存储、可见性、记忆作用域、审计

## 四大经验教训（Lessons）

- [[Lesson 1 公开工作]] - 信息透明，给予智能体广泛上下文
- [[Lesson 2 角色与工具]] - 明确角色定义，匹配工具
- [[Lesson 3 北极星]] - 设定团队北极星，激活智能体主动性
- [[Lesson 4 信任建立]] - 自主权与可靠性正相关，逐步扩展

## 关键模式

- [[Doer-Verifier 框架]] - 一个做、一个检查的双智能体架构
- [[生成器-评估器架构]] - GAN 启发的多轮迭代循环（Doer-Verifier 的工程实现深化）
- [[Harness 设计 - 长时应用开发]] - Planner + Generator + Evaluator 三智能体架构
- [[Skill Files 角色定义]] - Agent Skills 官方规范：SKILL.md 格式、三层渐进式披露

## 五个自检问题

建立人机协作团队前自问：

1. 信息是否公开、可搜索？→ [[Lesson 1 公开工作|Lesson 1]]
2. 能否写出花名册，明确每个角色？→ [[Lesson 2 角色与工具|Lesson 2]]
3. 每个人/智能体是否有合适工具？→ [[Lesson 2 角色与工具|Lesson 2]]
4. 是否有评估标准/测试？→ [[Lesson 4 信任建立|Lesson 4]]
5. 是否有清晰的北极星？→ [[Lesson 3 北极星|Lesson 3]]

## 来源

- **Building Effective Human-Agent Teams**: [[Building Effective Human-Agent Teams - Claude Blog]] (Kristen Swanson, Anthropic Education team)
- **官方 Agent 定义**: [[Anthropic Building Effective Agents - Engineering Blog]]
- **Agent Skills 规范**: [[Anthropic Equipping Agents with Skills]]
- **产品发布**: [[Introducing Claude Tag]]、[[What is Claude Tag]]
- **访问模型**: [[Agent identity a new access model for autonomous, team-wide AI]]
- **Harness 设计**: [[Harness design for long-running application development]]

---

*导航: 使用 Obsidian 图表视图查看整体结构*