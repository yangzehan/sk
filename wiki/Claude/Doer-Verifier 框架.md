---
title: Doer-Verifier 框架
type: pattern
created: 2026-06-25
source: "[[Building Effective Human-Agent Teams - Claude Blog]]"
related:
  - "[[构建有效的人机协作团队]]"
  - "[[Lesson 4 信任建立]]"
---

# Doer-Verifier 智能体框架

## 定义

**Doer-Verifier** 是 Anthropic 长期运行智能体的核心架构模式：将一个任务**拆分给两个智能体**——

- **Doer（执行者）**：负责完成任务
- **Verifier（验证者）**：负责检查 Doer 的产出

## 工作流程

```
任务输入
   │
   ▼
┌──────────┐    产出    ┌──────────┐
│   Doer   │ ────────→ │ Verifier │
└──────────┘           └──────────┘
                            │
                ┌───────────┼───────────┐
                ▼           ▼           ▼
            通过 ✅     打回重做 ❌     上报人类 ⚠️
```

## 为什么需要 Verifier？

### 单一智能体的局限性

- **既是运动员又是裁判**：自我审查存在盲区
- **难以发现自己的偏见**：同一种思路看不到同一种错误
- **缺乏交叉视角**：无法从不同立场审视同一产出

### Verifier 的价值

- **不同视角**：发现 Doer 看不到的问题
- **解耦审查与执行**：避免利益冲突
- **可独立训练**：Verifier 可以专精于某种验证标准

## 验证手段的多样性

最佳实践是给 Verifier 配置**多种验证手段**，不仅仅是"看一眼"：

| 工作类型 | 验证手段 |
|---------|---------|
| 代码 | 单元测试、集成测试、Lint、类型检查 |
| 技术文档 | 评分标准（rubric）、风格指南 |
| 设计稿 | 像素对比、组件库规范、可访问性 |
| 数据分析 | 数值校验、查询结果一致性 |
| 研究报告 | 来源校验、引用一致性 |

> 核心洞察：人类时间宝贵，**任何交给人类的成果都应已通过自动验证**。

## 适用场景

- 长期运行的任务（如代码重构、文档维护）
- 高风险产出（影响生产环境、对外发布）
- 智能体自主性较高、需要扩展自主权时

## 在 Anthropic 实践中的位置

Doer-Verifier 是 [[Lesson 4 信任建立|建立信任]] 阶段的关键工具——它让智能体在**不依赖人类实时审核**的情况下仍能保证质量，是自主权扩展的基础设施。

## 工程实现深化：生成器-评估器架构

> Anthropic 在 [Harness Design 工程博客](https://www.anthropic.com/engineering/harness-design-long-running-apps) 中给出了 Doer-Verifier 的**工程实现深化**——

详见 [[生成器-评估器架构]] 与 [[Harness 设计 - 长时应用开发]]，要点：

- 受 **GAN（生成对抗网络）** 启发，将"做"与"评"结构性分离
- 评估器使用 **Playwright MCP 实际交互**应用，而非仅静态检查
- 引入 **结构化评分标准**让主观质量可评分
- 多轮**迭代循环**（5-15 轮），每轮评估推动生成器演化
- 可选加入 **Planner** 形成三智能体架构
- **Sprint Contract 机制**：每个 sprint 开始前生成器与评估器**协商"完成"标准**
- **不是固定必要**——根据当前模型能力与任务边界动态决策

> 实测对比：构建 2D 复古游戏制作器——Solo 20 分钟 $9，游戏**坏了不能玩**；Full Harness 6 小时 $200，**能玩且功能丰富**。

## 来源

- **原始博客**: [[Building Effective Human-Agent Teams - Claude Blog]]
- **工程实现博客**: [[Harness design for long-running application development]]
- **相关页面**: [[生成器-评估器架构]]、[[Harness 设计 - 长时应用开发]]
- **官方链接**: <https://www.anthropic.com/engineering/harness-design-long-running-apps>