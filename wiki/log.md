# Wiki 日志

## [2026-06-25] ingest | Harness 设计 + Agent Identity + Claude Tag 产品文档

### 新增原始资料
- `原始资料/Harness design for long-running application development.md` - Anthropic Labs 的长时应用 Harness 设计博客
  - 涵盖: 朴素实现的两个失效模式（context anxiety + 自我评估偏差）、GAN 启发、前端设计 4 项评分标准、全栈开发三智能体架构、Sprint Contract、Playwright MCP、Opus 4.6 后的简化、成本对比

- `原始资料/Agent identity a new access model for autonomous, team-wide AI.md` - Claude Tag 的底层访问模型
  - 涵盖: 为什么"作为用户操作"不再可行、Claude 作为独立身份操作、权限继承模型、身份边界、DM 与频道的工具差异、安全与审计、未来方向

- `原始资料/Introducing Claude Tag.md` - Claude Tag 官方发布博客（2026-06-23）
  - 涵盖: 四种核心优势（多人、持续学习、主动汇报、异步）、启动方式、Anthropic 内部 65% 产品代码由 Claude Tag 生成

- `原始资料/What is Claude Tag.md` - Claude Tag 官方支持文档（2026-06-24）
  - 涵盖: 三种使用面、Member Access 三模式、Spend Limits 四控制、三层访问模型、数据可见性、记忆作用域、审计视图、数据删除路径

### 新增 Wiki 页面
- `wiki/Claude/Harness 设计 - 长时应用开发.md` - 长时应用 Harness 设计的完整工程解读
  - 涵盖: 朴素实现失效模式（context anxiety + 自我评估偏差）、Compaction vs Reset 对比、前端设计 4 项评分标准、Playwright MCP 评估循环、三智能体架构、Sprint Contract 机制、Opus 4.6 简化迭代、模型能力边界与评估器价值

- `wiki/Claude/生成器-评估器架构.md` - GAN 启发的多轮迭代循环模式
  - 涵盖: 核心原则（做评分离、主观质量可评分、精炼/切换战略）、评分标准设计模式、前端 vs 全栈两套工作循环、Sprint Contract、评估器调优、Evaluator 成本决策、逐一移除组件的简化方法

- `wiki/Claude/Agent Identity 访问模型.md` - Claude Tag 的访问模型
  - 涵盖: 为什么"作为用户操作"不可行、Claude 独立身份操作、权限继承模型、身份边界、广基线 + 精细覆盖建议、DM 与频道的工具差异、安全与审计、未来方向（JIT 凭证 + identity-aware overlay）

- `wiki/Claude/Claude Tag 设置与管理.md` - 管理员操作指南
  - 涵盖: 上线时间表（2026-08-03 强制切换）、三种使用面、四步启动、Member Access 三模式、Spend Limits 四控制、三层访问模型（Org → Workspace → Private Channel）、RBAC、管理员清单

- `wiki/Claude/Claude Tag 数据与记忆.md` - 数据存储、记忆与审计
  - 涵盖: 平台间数据隔离、数据存储与保留期、记忆作用域（Org/WS/Private）、审计视图与双重审计、频道内自检、数据删除两种路径

### 更新的 Wiki 页面
- `wiki/Claude/Claude Tag.md` - 全面重写
  - 整合官方发布博客与支持文档
  - 涵盖: 发布与可用性（Opus 4.8、2026-08-03 强制切换）、四种核心优势、内部影响（65% 产品代码）、接入四步流程、与 Agent Identity 关联

- `wiki/Claude/Doer-Verifier 框架.md` - 添加工程实现深化部分
  - 新增: 链接到生成器-评估器架构与 Harness 设计、简要介绍 GAN 启发、Playwright MCP、Sprint Contract、模型能力边界评估

- `wiki/Claude/INDEX.md` - 添加新条目
  - 新增: Agent Identity 访问模型、Claude Tag 设置与管理、Claude Tag 数据与记忆、生成器-评估器架构、Harness 设计 - 长时应用开发

### 更新的索引
- `wiki/INDEX.md` - 更新 Claude 主题描述
- `wiki/log.md` - 本条记录

---

## [2026-06-25] ingest | Anthropic 官方智能体定义与 Skill Files 规范

### 新增原始资料
- `原始资料/Anthropic Building Effective Agents - Engineering Blog.md` - Anthropic 智能体定义
  - 涵盖: Workflows vs Agents 架构区分、五大 Workflow 模式、智能体三大原则、ACI 设计建议、客户案例

- `原始资料/Anthropic Equipping Agents with Skills.md` - Agent Skills 技术规范
  - 涵盖: SKILL.md 格式、三层渐进式披露、Skill 工作流程、与代码执行的结合、最佳实践、安全考虑

### 新增 Wiki 页面
- `wiki/Claude/智能体官方定义.md` - Anthropic 对 Agent 的官方定义
  - 涵盖: 核心定义、Workflows vs Agents 区分、智能体核心特征、何时使用、五大 Workflow 模式、Multiplayer Agents 对比

### 更新的 Wiki 页面
- `wiki/Claude/Skill Files 角色定义.md` - 全面重写
  - 原博客仅一句话提及，现补充 Anthropic 官方工程博客的完整定义
  - 涵盖: SKILL.md YAML frontmatter、三层渐进式披露、PDF skill 示例、官方最佳实践、安全考虑、两个来源的对比关系

### 更新的索引
- `wiki/Claude/INDEX.md` - 新增"智能体官方定义"条目，Skill Files 描述更新


## [2026-06-25] ingest | Building Effective Human-Agent Teams

### 新增主题
- `wiki/Claude/` - Anthropic Claude 人机协作团队知识库

### 新增页面
- `wiki/Claude/INDEX.md` - 主题索引
- `wiki/Claude/构建有效的人机协作团队.md` - 四大经验教训总览
  - 来源: `原始资料/Building Effective Human-Agent Teams - Claude Blog.md`
  - 涵盖: 与传统 AI 协作的对比、四大经验、自检问题、关键洞见

- `wiki/Claude/Multiplayer Agents 概念.md` - 多人协作智能体定义
  - 涵盖: 持久记忆、独立凭证、持续广泛访问三大能力、与传统 Agent 对比

- `wiki/Claude/Claude Tag.md` - Anthropic 多人协作智能体产品
  - 涵盖: 核心定位、公开/私密两种使用场景、相关产品

- `wiki/Claude/Lesson 1 公开工作.md` - 信息透明与广泛上下文
  - 涵盖: 工作区级安全边界、默认公开、红利、文化转变代价

- `wiki/Claude/Lesson 2 角色与工具.md` - 明确角色与工具匹配
  - 涵盖: 角色多样性、智能体衍生、Skill Files 作用、清晰角色价值

- `wiki/Claude/Lesson 3 北极星.md` - 设定北极星激活智能体主动性
  - 涵盖: 主动提案权授予、真实案例、保护人类时间

- `wiki/Claude/Lesson 4 信任建立.md` - 自主权与可靠性正相关
  - 涵盖: 四项关键做法（多重验证、Doer-Verifier、反思错误、注意力稀缺原则）、500 bug 案例、backlog 治理四阶段

- `wiki/Claude/Doer-Verifier 框架.md` - 双智能体架构模式
  - 涵盖: 工作流程、单一智能体局限、验证手段多样性、适用场景

- `wiki/Claude/Skill Files 角色定义.md` - 标准化智能体角色
  - 涵盖: 定义、价值、内容模板、Anthropic 工程实践

### 原始资料
- `原始资料/Building Effective Human-Agent Teams - Claude Blog.md` - Anthropic 博客原文（英文+中文摘要）

### 更新的索引
- `wiki/INDEX.md` - 新增"Claude"主题
- `wiki/log.md` - 本条记录


## [2026-06-04] ingest | OpenCode 桌面端 /skills 弹窗

### 新增主题
- `wiki/opencode/` - OpenCode 项目知识库
  - 涵盖: Skills 系统、桌面端/TUI 架构对比、桌面端命令系统、本次实施记录

### 新增页面
- `wiki/opencode/INDEX.md` - 主题索引
- `wiki/opencode/架构/桌面端 vs TUI 架构对比.md` - 桌面端与 TUI 架构差异
  - 来源: `原始资料/opencode/01-skills-系统调查-代码路径与执行逻辑.md`
  - 涵盖: 命令系统、斜杠处理、弹窗组件对比

- `wiki/opencode/架构/桌面端命令系统.md` - 命令注册机制
  - 涵盖: CommandProvider、withCategory 工厂、Dialog 系统、List 组件

- `wiki/opencode/skills 系统/Skills 系统.md` - Skills 系统实现
  - 涵盖: 发现机制、注册为 command、API 端点、source 字段语义

- `wiki/opencode/实践/实施记录 - 桌面端 /skills 弹窗.md` - 本次实施记录
  - 来源: `原始资料/opencode/02-实施计划与设计决策.md`
  - 涵盖: 改动清单、关键设计、验证结果、边缘情况、手动测试用例

### 原始资料
- `原始资料/opencode/01-skills-系统调查-代码路径与执行逻辑.md` - 代码路径 + 执行逻辑
- `原始资料/opencode/02-实施计划与设计决策.md` - 计划文件 + 决策点

### 更新的索引
- `wiki/INDEX.md` - 新增"opencode"主题
- `wiki/log.md` - 本条记录

### 实施结果
- 新建文件 1 个（dialog-select-skill.tsx）
- 修改文件 19 个（use-session-commands.tsx、prompt-input.tsx、18 个 i18n）
- 验证: typecheck ✅、parity test ✅（1 pass / 68 expects）、unit tests ✅（339 pass / 0 fail）


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