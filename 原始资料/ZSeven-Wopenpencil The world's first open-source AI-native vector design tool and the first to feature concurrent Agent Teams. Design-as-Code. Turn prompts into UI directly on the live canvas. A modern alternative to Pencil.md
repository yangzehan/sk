---
title: "ZSeven-W/openpencil: The world's first open-source AI-native vector design tool and the first to feature concurrent Agent Teams. Design-as-Code. Turn prompts into UI directly on the live canvas. A modern alternative to Pencil."
source: "https://github.com/ZSeven-W/openpencil"
author:
published:
created: 2026-04-07
description: "The world's first open-source AI-native vector design tool and the first to feature concurrent Agent Teams. Design-as-Code. Turn prompts into UI directly on the live canvas. A modern alternative to Pencil. - ZSeven-W/openpencil"
tags:
  - "clippings"
---
[![[f00ab43b4d27fcd2865589fc8c7b76f1_MD5.jpg]]](https://github.com/ZSeven-W/openpencil/blob/main/apps/desktop/build/icon.png)

## OpenPencil

**The world's first open-source AI-native vector design tool.**  
<sub>Concurrent Agent Teams • Design-as-Code • Built-in MCP Server • Multi-model Intelligence</sub>

[**English**](https://github.com/ZSeven-W/openpencil/blob/main/README.md) · [简体中文](https://github.com/ZSeven-W/openpencil/blob/main/README.zh.md) · [繁體中文](https://github.com/ZSeven-W/openpencil/blob/main/README.zh-TW.md) · [日本語](https://github.com/ZSeven-W/openpencil/blob/main/README.ja.md) · [한국어](https://github.com/ZSeven-W/openpencil/blob/main/README.ko.md) · [Français](https://github.com/ZSeven-W/openpencil/blob/main/README.fr.md) · [Español](https://github.com/ZSeven-W/openpencil/blob/main/README.es.md) · [Deutsch](https://github.com/ZSeven-W/openpencil/blob/main/README.de.md) · [Português](https://github.com/ZSeven-W/openpencil/blob/main/README.pt.md) · [Русский](https://github.com/ZSeven-W/openpencil/blob/main/README.ru.md) · [हिन्दी](https://github.com/ZSeven-W/openpencil/blob/main/README.hi.md) · [Türkçe](https://github.com/ZSeven-W/openpencil/blob/main/README.tr.md) · [ไทย](https://github.com/ZSeven-W/openpencil/blob/main/README.th.md) · [Tiếng Việt](https://github.com/ZSeven-W/openpencil/blob/main/README.vi.md) · [Bahasa Indonesia](https://github.com/ZSeven-W/openpencil/blob/main/README.id.md)

  

[![[0f06e59710796d1a59ecd5fad78eea4e_MD5.jpg]]](https://oss.ioa.tech/zseven/openpencil/a46e24733239ce24de36702342201033.mp4)

<sub>Click the image to watch the demo video</sub>

  

> **Note:** There is another open-source project with the same name — [OpenPencil](https://github.com/open-pencil/open-pencil), focused on Figma-compatible visual design with real-time collaboration. This project focuses on AI-native design-to-code workflows.

## Why OpenPencil

| ### 🎨 Prompt → Canvas  Describe any UI in natural language. Watch it appear on the infinite canvas in real-time with streaming animation. Modify existing designs by selecting elements and chatting. | ### 🤖 Concurrent Agent Teams  The orchestrator decomposes complex pages into spatial sub-tasks. Multiple AI agents work on different sections simultaneously — hero, features, footer — all streaming in parallel. |
| --- | --- |
| ### 🧠 Multi-Model Intelligence  Automatically adapts to each model's capabilities. Claude gets full prompts with thinking; GPT-4o/Gemini disable thinking; smaller models (MiniMax, Qwen, Llama) get simplified prompts for reliable output. | ### 🔌 MCP Server  One-click install into Claude Code, Codex, Gemini, OpenCode, Kiro, or Copilot CLIs. Design from your terminal — read, create, and modify `.op` files through any MCP-compatible agent. |
| ### 📦 Design-as-Code  `.op` files are JSON — human-readable, Git-friendly, diffable. Design variables generate CSS custom properties. Code export to React + Tailwind or HTML + CSS. | ### 🖥️ Runs Everywhere  Web app + native desktop on macOS, Windows, and Linux via Electron. Auto-updates from GitHub Releases. `.op` file association — double-click to open. |
| ### ⌨️ CLI — op  Control the design tool from your terminal. `op design`, `op insert`, `op export` — batch design DSL, node manipulation, code export. Pipe in from files or stdin. Works with desktop app or web server. | ### 🎯 Multi-Platform Code Export  Export to React + Tailwind, HTML + CSS, Vue, Svelte, Flutter, SwiftUI, Jetpack Compose, React Native — all from one `.op` file. Design variables become CSS custom properties. |

## Install

**macOS (Homebrew):**

```
brew tap zseven-w/openpencil
brew install --cask openpencil
```

**Windows (Scoop):**

```
scoop bucket add openpencil https://github.com/zseven-w/scoop-openpencil
scoop install openpencil
```

**Linux / Windows direct download:** [GitHub Releases](https://github.com/ZSeven-W/openpencil/releases) — `.exe` (Windows), `.AppImage` / `.deb` (Linux)

**CLI (`op`):**

```
npm install -g @zseven-w/openpencil
```

## Quick Start (Development)

```
# Install dependencies
bun install

# Start dev server at http://localhost:3000
bun --bun run dev
```

Or run as a desktop app:

```
bun run electron:dev
```

> **Prerequisites:** [Bun](https://bun.sh/) >= 1.0 and [Node.js](https://nodejs.org/) >= 18

### Docker

Multiple image variants are available — pick the one that fits your needs:

| Image | Size | Includes |
| --- | --- | --- |
| `openpencil:latest` | ~226 MB | Web app only |
| `openpencil-claude:latest` | — | \+ Claude Code CLI |
| `openpencil-codex:latest` | — | \+ Codex CLI |
| `openpencil-opencode:latest` | — | \+ OpenCode CLI |
| `openpencil-copilot:latest` | — | \+ GitHub Copilot CLI |
| `openpencil-gemini:latest` | — | \+ Gemini CLI |
| `openpencil-full:latest` | ~1 GB | All CLI tools |

**Run (web only):**

```
docker run -d -p 3000:3000 ghcr.io/zseven-w/openpencil:latest
```

**Run with AI CLI (e.g. Claude Code):**

The AI chat relies on Claude CLI OAuth login. Use a Docker volume to persist the login session:

```
# Step 1 — Login (one-time)
docker volume create openpencil-claude-auth
docker run -it --rm \
  -v openpencil-claude-auth:/root/.claude \
  ghcr.io/zseven-w/openpencil-claude:latest claude login

# Step 2 — Start
docker run -d -p 3000:3000 \
  -v openpencil-claude-auth:/root/.claude \
  ghcr.io/zseven-w/openpencil-claude:latest
```

**Build locally:**

```
# Base (web only)
docker build --target base -t openpencil .

# With a specific CLI
docker build --target with-claude -t openpencil-claude .

# Full (all CLIs)
docker build --target full -t openpencil-full .
```

## AI-Native Design

**Prompt to UI**

- **Text-to-design** — describe a page, get it generated on canvas in real-time with streaming animation
- **Orchestrator** — decomposes complex pages into spatial sub-tasks for parallel generation
- **Design modification** — select elements, then describe changes in natural language
- **Vision input** — attach screenshots or mockups for reference-based design

**Multi-Agent Support**

| Agent | Setup |
| --- | --- |
| **Built-in (9+ providers)** | Select from provider presets with region switcher — Anthropic, OpenAI, Google, DeepSeek, and more |
| **Claude Code** | No config — uses Claude Agent SDK with local OAuth |
| **Codex CLI** | Connect in Agent Settings (`Cmd+,`) |
| **OpenCode** | Connect in Agent Settings (`Cmd+,`) |
| **GitHub Copilot** | `copilot login` then connect in Agent Settings (`Cmd+,`) |
| **Gemini CLI** | Connect in Agent Settings (`Cmd+,`) |

**Model Capability Profiles** — automatically adapts prompts, thinking mode, and timeouts per model tier. Full-tier models (Claude) get complete prompts; standard-tier (GPT-4o, Gemini, DeepSeek) disable thinking; basic-tier (MiniMax, Qwen, Llama, Mistral) get simplified nested-JSON prompts for maximum reliability.

**i18n** — Full interface localization in 15 languages: English, 简体中文, 繁體中文, 日本語, 한국어, Français, Español, Deutsch, Português, Русский, हिन्दी, Türkçe, ไทย, Tiếng Việt, Bahasa Indonesia.

**MCP Server**

- Built-in MCP server — one-click install into Claude Code / Codex / Gemini / OpenCode / Kiro / Copilot CLIs
- Auto-detects Node.js — if not installed, falls back to HTTP transport and auto-starts the MCP HTTP server
- Design automation from terminal: read, create, and modify `.op` files via any MCP-compatible agent
- **Layered design workflow** — `design_skeleton` → `design_content` → `design_refine` for higher-fidelity multi-section designs
- **Segmented prompt retrieval** — load only the design knowledge you need (schema, layout, roles, icons, planning, etc.)
- Multi-page support — create, rename, reorder, and duplicate pages via MCP tools

**Code Generation**

- React + Tailwind CSS, HTML + CSS, CSS Variables
- Vue, Svelte, Flutter, SwiftUI, Jetpack Compose, React Native

## CLI — op

Install globally and control the design tool from your terminal:

```
npm install -g @zseven-w/openpencil
```
```
op start                     # Launch desktop app
op design @landing.txt       # Batch design from file
op insert '{"type":"RECT"}'  # Insert a node
op export react --out .      # Export to React + Tailwind
op import:figma design.fig   # Import Figma file
cat design.dsl | op design - # Pipe from stdin
```

Supports three input methods: inline string, `@filepath` (read from file), or `-` (read from stdin). Works with desktop app or web dev server. See [CLI README](https://github.com/ZSeven-W/openpencil/blob/main/apps/cli/README.md) for full command reference.

**LLM Skill** — install the [OpenPencil Skill](https://github.com/ZSeven-W/openpencil-skill) plugin to teach AI agents (Claude Code, Cursor, Codex, Gemini CLI, etc.) how to design with `op`.

## Features

**Canvas & Drawing**

- Infinite canvas with pan, zoom, smart alignment guides, and snapping
- Rectangle, Ellipse, Line, Polygon, Pen (Bezier), Frame, Text
- Boolean operations — union, subtract, intersect with contextual toolbar
- Icon picker (Iconify) and image import (PNG/JPEG/SVG/WebP/GIF)
- Auto-layout — vertical/horizontal with gap, padding, justify, align
- Multi-page documents with tab navigation

**Design System**

- Design variables — color, number, string tokens with `$variable` references
- Multi-theme support — multiple axes, each with variants (Light/Dark, Compact/Comfortable)
- Component system — reusable components with instances and overrides
- CSS sync — auto-generated custom properties, `var(--name)` in code output

**Figma Import**

- Import `.fig` files with layout, fills, strokes, effects, text, images, and vectors preserved

**Desktop App**

- Native macOS, Windows, and Linux via Electron
- `.op` file association — double-click to open, single-instance lock
- Auto-update from GitHub Releases
- Native application menu and file dialogs

## Tech Stack

|  |  |
| --- | --- |
| **Frontend** | React 19 · TanStack Start · Tailwind CSS v4 · shadcn/ui · i18next |
| **Canvas** | CanvasKit/Skia (WASM, GPU-accelerated) |
| **State** | Zustand v5 |
| **Server** | Nitro |
| **Desktop** | Electron 35 |
| **CLI** | `op` — terminal control, batch design DSL, code export |
| **AI** | Vercel AI SDK v6 · Anthropic SDK · Claude Agent SDK · OpenCode SDK · Copilot SDK |
| **Runtime** | Bun · Vite 7 |
| **File format** | `.op` — JSON-based, human-readable, Git-friendly |

## Project Structure

```
openpencil/
├── apps/
│   ├── web/                 TanStack Start web app
│   │   ├── src/
│   │   │   ├── canvas/      CanvasKit/Skia engine — drawing, sync, layout
│   │   │   ├── components/  React UI — editor, panels, shared dialogs, icons
│   │   │   ├── services/ai/ AI chat, orchestrator, design generation, streaming
│   │   │   ├── services/codegen/ Code generation service wrappers
│   │   │   ├── stores/      Zustand — canvas, document, pages, history, AI
│   │   │   ├── mcp/         MCP server tools for external CLI integration
│   │   │   ├── hooks/       Keyboard shortcuts, file drop, Figma paste, MCP sync
│   │   │   ├── i18n/        Internationalization — 15 locales
│   │   │   └── uikit/       Reusable component kit system
│   │   └── server/
│   │       ├── api/ai/      Nitro API — streaming chat, agent, generation, image search
│   │       ├── api/mcp/     MCP HTTP transport endpoints
│   │       └── utils/       Claude, OpenCode, Codex, Copilot, Gemini CLI wrappers
│   ├── desktop/             Electron desktop app
│   │   ├── main.ts          Window, Nitro fork, native menu, auto-updater
│   │   ├── ipc-handlers.ts  Native file dialogs, theme sync, prefs IPC
│   │   └── preload.ts       IPC bridge
│   └── cli/                 CLI tool — \`op\` command
│       ├── src/commands/    Design, document, export, import, node, page, variable commands
│       ├── connection.ts    WebSocket connection to running app
│       └── launcher.ts      Auto-detect and launch desktop app or web server
├── packages/
│   ├── pen-types/           Type definitions for PenDocument model
│   ├── pen-core/            Document tree ops, layout engine, variables
│   ├── pen-codegen/         Code generators (React, HTML, Vue, Flutter, ...)
│   ├── pen-figma/           Figma .fig file parser and converter
│   ├── pen-renderer/        Standalone CanvasKit/Skia renderer
│   ├── pen-sdk/             Umbrella SDK (re-exports all packages)
│   ├── pen-ai-skills/       AI prompt skill engine (phase-driven prompt loading)
│   └── agent/               AI agent SDK (Vercel AI SDK, multi-provider, agent teams)
└── .githooks/               Pre-commit version sync from branch name
```

## Keyboard Shortcuts

| Key | Action |  | Key | Action |
| --- | --- | --- | --- | --- |
| `V` | Select |  | `Cmd+S` | Save |
| `R` | Rectangle |  | `Cmd+Z` | Undo |
| `O` | Ellipse |  | `Cmd+Shift+Z` | Redo |
| `L` | Line |  | `Cmd+C/X/V/D` | Copy/Cut/Paste/Duplicate |
| `T` | Text |  | `Cmd+G` | Group |
| `F` | Frame |  | `Cmd+Shift+G` | Ungroup |
| `P` | Pen tool |  | `Cmd+Shift+E` | Export |
| `H` | Hand (pan) |  | `Cmd+Shift+C` | Code panel |
| `Del` | Delete |  | `Cmd+Shift+V` | Variables panel |
| `[ / ]` | Reorder |  | `Cmd+J` | AI chat |
| Arrows | Nudge 1px |  | `Cmd+,` | Agent settings |
| `Cmd+Alt+U` | Boolean union |  | `Cmd+Alt+S` | Boolean subtract |
| `Cmd+Alt+I` | Boolean intersect |  |  |  |

## Scripts

```
bun --bun run dev          # Dev server (port 3000)
bun --bun run build        # Production build
bun --bun run test         # Run tests (Vitest)
npx tsc --noEmit           # Type check
bun run bump <version>     # Sync version across all package.json
bun run electron:dev       # Electron dev
bun run electron:build     # Electron package
bun run cli:dev            # Run CLI from source
bun run cli:compile        # Compile CLI to dist
```

## Contributing

Contributions are welcome! See [CLAUDE.md](https://github.com/ZSeven-W/openpencil/blob/main/CLAUDE.md) for architecture details and code style.

1. Fork and clone
2. Set up version sync: `git config core.hooksPath .githooks`
3. Create a branch: `git checkout -b feat/my-feature`
4. Run checks: `npx tsc --noEmit && bun --bun run test`
5. Commit with [Conventional Commits](https://www.conventionalcommits.org/): `feat(canvas): add rotation snapping`
6. Open a PR against `main`

## Roadmap

- Design variables & tokens with CSS sync
- Component system (instances & overrides)
- AI design generation with orchestrator
- MCP server integration with layered design workflow
- Multi-page support
- Figma `.fig` import
- Boolean operations (union, subtract, intersect)
- Multi-model capability profiles
- Monorepo restructure with reusable packages
- CLI tool (`op`) for terminal control
- Built-in AI agent SDK with multi-provider support
- i18n — 15 languages
- Collaborative editing
- Plugin system

## Contributors

[![[98aa446467a450056dc9f0ae63fa0155_MD5.svg]]](https://github.com/ZSeven-W/openpencil/graphs/contributors)

## Community

[**Join our Discord**](https://discord.gg/h9Fmyy6pVh) — Ask questions, share designs, suggest features.

## Star History

[![[ff750a38ed3d1a0eb2153d3eb923c09b_MD5.svg]]](https://star-history.com/#ZSeven-W/openpencil&Date)

## License

[MIT](https://github.com/ZSeven-W/openpencil/blob/main/LICENSE) — Copyright (c) 2026 ZSeven-W