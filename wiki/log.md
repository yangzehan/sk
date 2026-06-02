# Wiki 日志

## [2026-05-25] 目录结构重组

### 新结构

```
wiki/
├── Codex/               # Codex 相关
│   ├── 参考/             # 协议、Schema、CLI
│   ├── 模式/             # 技能、MCP、文件系统、顾问策略
│   └── 实践/             # OpenPencil
├── LLM-Wiki/             # LLM Wiki 方法论
└── 实践/                 # 故障排查
```

### 移动的文件

**Codex/参考/**
- 通信协议.md (原 Protocol.md)
- 核心原语.md (原 Core Primitives.md)
- 生命周期.md (原 Lifecycle.md)
- Codex-CLI.md
- Codex-Schema-索引.md
- Thread.md, Turn.md, ThreadItem.md

**Codex/模式/**
- 技能系统.md (原 Skills.md)
- MCP-集成.md (原 MCP Integration.md)
- 文件系统.md (原 File System.md)
- 命令执行.md (原 Command Execution.md)
- 顾问策略.md (原 Advisor Strategy.md)

**Codex/实践/**
- OpenPencil.md

**LLM-Wiki/**
- 核心理念.md (原 LLM Wiki Philosophy.md)
- 架构.md (原 LLM Wiki Architecture.md)
- 运营.md (原 LLM Wiki Operations.md)

**实践/**
- ClickHouse TOO_MANY_PARTS 故障排查.md


## [2026-04-10] ingest | Advisor Strategy & OpenPencil

### 新增页面

- `wiki/Codex/模式/顾问策略.md` - Agent 顾问架构模式文档
  - 来源: `原始资料/Advisor tool.md`, `原始资料/The advisor strategy Give Sonnet an intelligence boost with Opus.md`
  - 涵盖: 架构理念、模型兼容性、性能数据、实现方式、最佳实践

- `wiki/Codex/实践/OpenPencil.md` - AI 原生设计工具文档
  - 来源: `原始资料/ZSeven-Wopenpencil The world's first open-source AI-native vector design tool...`
  - 涵盖: 核心特性、技术栈、MCP 集成、代码导出

### 更新的索引

- `wiki/Codex/INDEX.md` - 新增 Advisor Strategy 和 OpenPencil 条目
- `wiki/INDEX.md` - 新增"新增 (2026-04-10)"部分


## [2026-05-25] ingest | ClickHouse TOO_MANY_PARTS 故障排查

### 新增页面
- `wiki/实践/ClickHouse TOO_MANY_PARTS 故障排查.md`
  - 来源: `troubleshooting-portrait_tag_0_backup_local-TOO_MANY_PARTS.md`
  - 涵盖: 根因分析（双字段分区键导致笛卡尔积）、三套解决方案、关键参数说明、验证命令

### 更新的索引
- `wiki/实践/INDEX.md` - 新增 ClickHouse TOO_MANY_PARTS 故障排查条目