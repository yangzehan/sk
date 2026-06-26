---
title: Knowledge Wiki Index
type: index
created: 2026-04-06
updated: 2026-06-26
---

# Knowledge Wiki Index

## 知识分类

### [[Codex/INDEX|Codex]]
Codex App-Server 技术文档

### [[LLM-Wiki/INDEX|LLM-Wiki]]
LLM Wiki 方法论 - Karpathy 的知识库模式

### [[Claude/INDEX|Claude]]
Anthropic Claude — 人机协作团队、Multiplayer Agents、Claude Tag

### [[pi-hermes-memory/INDEX|pi-hermes-memory]]
Pi 编码 Agent 的持久化记忆插件（chandra447/pi-hermes-memory）—— 注入机制、配置项、后台处理器、子进程设计

### [[实践/INDEX|实践]]
故障排查与实战记录

### [[ClickHouse/INDEX|ClickHouse]]
ClickHouse 知识库 — 架构、升级迁移、脚本工具

### [[opencode/INDEX|opencode]]
OpenCode 项目 — Skills 系统、桌面端/TUI 架构对比、命令系统

---

## 新增 (2026-06-26)

- `wiki/pi-hermes-memory/` 主题新建
  - 来源: 本次会话（2026-06-26）对本地安装的 `pi-hermes-memory@0.7.20` 源码逐行解析
  - 涵盖: 插件定位、9 命令 + 4 工具、28 配置项、`memoryMode` / `memoryPolicyStyle` 注入规则、`<memory-context>` fence、frozen snapshot、background review / session flush / correction detection 三大处理器、auto-consolidate 子进程设计（`pi -p --no-session --no-extensions`）、三种 overflow 策略（`auto-consolidate` / `reject` / `fifo-evict`）、correction 2-pass 文本匹配、事件生命周期对比

---

## 新增 (2026-06-25)

- `wiki/Claude/` 主题扩展
  - 来源: `原始资料/Building Effective Human-Agent Teams - Claude Blog.md`、`原始资料/Anthropic Building Effective Agents - Engineering Blog.md`、`原始资料/Anthropic Equipping Agents with Skills.md`、`原始资料/Introducing Claude Tag.md`、`原始资料/What is Claude Tag.md`、`原始资料/Agent identity a new access model for autonomous, team-wide AI.md`、`原始资料/Harness design for long-running application development.md`
  - 涵盖: 四大经验教训、Multiplayer Agents 概念、智能体官方定义、Agent Identity 访问模型、Claude Tag 产品 + 设置与数据/记忆、Doer-Verifier + 生成器-评估器架构、Harness 设计、Skill Files 规范

---

## 新增 (2026-06-04)

- `wiki/opencode/` 主题新建
  - 涵盖: 桌面端 vs TUI 架构对比、Skills 系统、桌面端命令系统、本次 /skills 弹窗实施记录
  - 来源: 本次会话（2026-06-04）opencode 桌面端 /skills 弹窗实施

---

*导航: 使用 Obsidian 图表视图查看整体结构*