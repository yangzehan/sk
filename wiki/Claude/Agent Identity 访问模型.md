---
title: Agent Identity 访问模型
type: concept
created: 2026-06-25
source: "[[Agent identity a new access model for autonomous, team-wide AI]]"
related:
  - "[[Claude Tag]]"
  - "[[Claude Tag 设置与管理]]"
  - "[[Multiplayer Agents 概念]]"
  - "[[构建有效的人机协作团队]]"
  - "[[智能体官方定义]]"
---

# Agent Identity 访问模型

> **核心定义**：在 Multiplayer AI 场景中，**Claude 不再"作为用户"操作，而是拥有自己独立的账户和凭证**，由管理员配置并绑定到工作空间（workspace）。

这是 [[Claude Tag]] 的**底层访问模型**，也是 Multiplayer Agents 从"借用人类账号"演化为"独立团队成员"的关键基础设施。

> **官方表述**："Agent identity"（智能体身份）是一种新的访问模型——它把权限从"**按用户**"迁移到"**按渠道/空间**"。

---

## 为什么"作为用户操作"不再可行

在单人 AI 场景中（个人聊天、个人助手），让 AI **借用你的账号**操作 Google Drive、GitHub、日历是合理的。但在 Multiplayer 场景中这会**根本性失效**，原因有二：

### 1. 智能体自主性持续增强

> "The length of a task that an AI agent can reliably complete on its own has been **doubling roughly every four months**."

智能体能可靠完成的任务长度**约每四个月翻一番**。它们：

- 会**自己安排未来任务**
- 在原始提问者**早已下线**后仍持续响应事件
- 在"无人驾驶"状态下长期运行

——这种模式下，**借用某个用户的账号**明显不合适。

### 2. Multiplayer 团队场景

> Claude Tag 把 Claude 放进**共享空间**——例如 3 个工程师 + 1 个 PM 在一起 debug 的频道。

但当**多个人同时在指挥**时：

- **该用谁的权限？** 没有永远正确的答案
- 需要能**独立于人类**定义 Claude 在 Slack 中能做什么
- 需要**独立追踪** Claude 在 Slack 中的所有动作

---

## Claude 作为"它自己"操作

在一个启用了 Claude Tag 的频道中：

| 操作 | 由谁执行 |
|------|---------|
| 在 Slack 中发言 | **Claude App**（独立账号） |
| 发起 PR | **Claude GitHub App**（独立身份） |
| 查询数据仓库 | **管理员配置的服务账号** |

> **关键安全收益**：由于不涉及个人凭证，**共享频道永远不会成为通往某人私密文档的"侧门"**。

---

## 权限继承模型（Inheriting Permissions）

管理员在 **workspace 级别**定义一份**基线身份**（baseline identity）——基线包括 Claude 在所有地方共享的连接和技能。

默认情况下，**每个频道都继承**这份基线。然后按需**在频道级别覆盖**：

```
Workspace（基线身份）
├── 公共频道 #general        → 继承基线（基础工具）
├── 公共频道 #engineering    → 覆盖：增加 GitHub + 数据仓库
├── 公共频道 #data-warehouse → 覆盖：增加只读仓库权限
└── 私有频道 #crm-secret     → 覆盖：仅该频道可见 CRM 连接
```

管理员除了凭证，还能定义：

- **Repository access**：Claude 可读/写的代码仓库
- **Connectors**：工具和 API key；同一服务在不同频道可使用**不同权限级别**的 key
  > 例：通用频道对仓库**只读**，数据团队私有频道对仓库**可写**
- **Skills and plugins**：Claude 可动态加载的指令、脚本、资源文件夹
- **Standing instructions**：每个频道的自定义指令和上下文

> **可逆性优势**：因为身份独立存在，**撤销这个身份就立刻终结 Claude 在所有地方用它的访问**——比逐个审计数十个用户账号下的智能体操作**管理成本低一个数量级**。

---

## 身份边界如何工作

> **核心转变**：把"**这个用户能做什么？**"换成"**这个智能体在这个 compartment（隔离空间）里能做什么？**"

这是对**传统按用户 ACL（访问控制列表）**的根本性偏离。

### 具体表现

Claude Tag 为**每个私有频道**创建独立的智能体身份；工作区内的**公共频道共享工作区级身份**：

| 场景 | 行为 |
|------|------|
| 法务频道的 Claude | **不能触达**未授予的代码 |
| 工程频道的 Claude | **不能读取**未授予的法务文档 |
| 私有频道学到的内容 | **永远不会出现在**更广的工作区 |

### 关键属性

- **身份归属于频道**：频道中任何人默认可 @Claude
- **最小权限原则**：管理员可将每个频道配置为"**最低权限成员**"的权限
- **Enterprise RBAC**：可进一步限制"**谁能调用 Claude**"

> **看似反直觉**：一个**没有仓库直接访问权限**的频道成员，**仍可让 Claude 读取该仓库**——只要该频道的身份被授予了此权限。

---

## 工具与上下文的"广基线 + 精细覆盖"

> [!tip] 核心建议
> **从慷慨的基线开始**，然后根据组织管理偏好逐步收紧。

### 为什么"广基线"

Anthropic 内部运行 Claude Tag 时发现：

> **它的价值随工具和上下文访问的扩展而**复利式增长**——每一个新连接的系统都让所有其他系统更有用，因为 Claude 能跨系统组合上下文。**

Claude 可以把：

- Slack 中的线索
- Drive 中的文档
- Tracker 中的工单
- 数据仓库中的查询

——**组合成一个答案**，这是任何单一工具都做不到的。

### 实操建议

1. 在**几个频道**设置基线身份
2. **阅读审计日志**
3. 每次只**有意识地**增加一个授权

### 更细粒度的控制

对组织要求更细的：

- 可在**特定频道禁用** Claude Tag
- 应用 **RBAC** 限制特定用户才能访问 Claude Tag

---

## Direct Messages（直接消息）

| 场景 | 执行身份 | 适用 |
|------|---------|------|
| 共享频道中的 @Claude | **组织身份**（workspace identity） | 团队工作 |
| DM 中的 @Claude | **用户个人** claude.ai 账户 | 个人敏感任务 |

> DM 使用用户的**个人连接器、凭证**，**名字也会出现在结果中**。适合处理**永远不应进频道**的任务，例如：
> - 邮件草稿
> - 只有你才有许可证的软件

---

## 安全与审计

### 凭证注入机制

管理员向频道身份**添加连接**时：

1. 凭证被**独立存储**
2. **映射**到该频道身份
3. 在**网络边界**于请求时**注入**

> **强制出站规则**：**任何管理员未允许的主机**的出站流量**会被直接阻断**。

### 审计覆盖范围

每次使用智能体凭证执行的操作都会被记录：

- 调度的任务（routines）
- 记忆写入（memory writes）
- 网络调用（network calls）
- 每个连接系统的**自身日志**（因为 Claude 在其服务账号下操作）

---

## 未来方向

Anthropic 计划加强的安全能力：

1. **Just-in-time 凭证授予**
   - 用户可在**当下批准单个敏感动作**，无需永久扩大智能体权限范围

2. **Identity-aware 覆盖层**
   - 适用于有更复杂权限结构的组织
   - 在智能体作用域之上**叠加用户级检查**
   - Claude 只有在**频道身份**和**请求用户本人权限**都允许时才执行

---

## 与 Multiplayer Agents 三大能力的关系

| Multiplayer 能力 | Agent Identity 的实现 |
|----------------|---------------------|
| 持久记忆 | 记忆按**频道/工作区**作用域存储 |
| 独立凭证 | **就是 Agent Identity 的核心** |
| 持续广泛访问 | workspace 级基线 + 频道级覆盖 |

---

## 来源

- **官方博客原文**: [[Agent identity a new access model for autonomous, team-wide AI]]
- **作者**: Noah Zweben（Anthropic Claude Code 团队）
- **产品**: [[Claude Tag]]
- **相关博客**: [[Introducing Claude Tag]]、[[Building Effective Human-Agent Teams - Claude Blog]]
- **官方链接**: <https://claude.com/blog/agent-identity-access-model>