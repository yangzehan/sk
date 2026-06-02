---
title: OpenPencil
type: tool
created: 2026-04-07
source: 原始资料/ZSeven-Wopenpencil The world's first open-source AI-native vector design tool and the first to feature concurrent Agent Teams. Design-as-Code. Turn prompts into UI directly on the live canvas. A modern alternative to Pencil.md
tags:
  - design-tool
  - ai-agent
  - mcp
  - open-source
---

# OpenPencil

开源的 AI 原生矢量设计工具，首个支持并发 Agent 团队的设计平台。

## 核心特性

| 特性 | 说明 |
|------|------|
| **Prompt → Canvas** | 自然语言描述 UI，实时流式生成到无限画布 |
| **并发 Agent 团队** | 编排器将复杂页面分解为空间子任务，多个 AI Agent 并行工作 |
| **多模型智能** | 自动适应各模型能力：Claude 获完整提示词，GPT-4o/Gemini 禁用思考，小模型获简化提示 |
| **MCP 服务器** | 一键安装到 Claude Code、Codex、 Gemini、OpenCode 等 Agent CLI |
| **Design-as-Code** | `.op` 文件为 JSON 格式，人类可读、Git 友好、可 diff |

## 技术栈

| 层 | 技术 |
|----|------|
| 前端 | React 19 · TanStack Start · Tailwind CSS v4 |
| 画布引擎 | CanvasKit/Skia (WASM, GPU 加速) |
| 状态管理 | Zustand v5 |
| 服务器 | Nitro |
| 桌面端 | Electron 35 |
| AI | Vercel AI SDK · Anthropic SDK · Claude Agent SDK |
| 文件格式 | `.op` (JSON-based) |

## 与 Agent 的集成

OpenPencil 内置 MCP 服务器，可与多种 Agent CLI 集成：

- **Claude Code**: 使用 Claude Agent SDK，本地 OAuth 登录
- **Codex CLI**: 通过 Agent Settings 连接
- **OpenCode**: 通过 Agent Settings 连接
- **GitHub Copilot**: `copilot login` 后连接
- **Gemini CLI**: 通过 Agent Settings 连接

### MCP 工作流

1. `design_skeleton` - 创建设计骨架
2. `design_content` - 填充内容
3. `design_refine` - 细化设计

## 代码导出

支持多平台导出：
- React + Tailwind CSS
- HTML + CSS
- Vue, Svelte
- Flutter, SwiftUI, Jetpack Compose
- React Native

设计变量自动转换为 CSS 自定义属性。

## 相关资源

- [[顾问策略]] - Agent 架构模式
- [[架构|LLM Wiki 架构]] - 知识管理
