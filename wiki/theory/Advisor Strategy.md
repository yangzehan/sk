---
title: Advisor Strategy
type: concept
created: 2026-04-10
source: raw/Advisor tool.md, raw/The advisor strategy Give Sonnet an intelligence boost with Opus.md
tags:
  - agent-pattern
  - claude
  - cost-optimization
---

# Advisor Strategy（顾问策略）

一种将更强大模型（如 Opus）作为战术顾问，与更快速、成本更低的执行者模型（如 Sonnet 或 Haiku）配对的 Agent 架构模式。

## 核心理念

在顾问策略中，执行者（Sonnet/Haiku）端到端运行任务：调用工具、读取结果、迭代寻找解决方案。当执行者遇到无法合理解决的决策时，它会咨询 Opus 作为顾问获取指导。Opus 访问共享上下文并返回计划、修正或停止信号，然后执行者继续执行。

顾问从不调用工具，也不生成面向用户的输出，只为执行者提供指导。

## 架构优势

| 对比项 | 传统子代理模式 | 顾问策略 |
|--------|----------------|----------|
| 编排方式 | 大模型分解任务并委派给工作者 | 小模型驱动并按需升级到顾问 |
| 推理成本 | 全部端到端使用大模型 | 仅在需要时升级到 Opus |
| 复杂性 | 需要分解、员工池、编排逻辑 | 无需分解，执行者自动判断何时升级 |

## 模型兼容性

| 执行者模型 | 顾问模型 |
|------------|----------|
| Claude Haiku 4.5 | Claude Opus 4.6 |
| Claude Sonnet 4.6 | Claude Opus 4.6 |
| Claude Opus 4.6 | Claude Opus 4.6 |

## 性能表现

- **SWE-bench 多语言**: Sonnet + Opus 顾问 **11.9%** 提升，每任务成本降低
- **BrowseComp**: Haiku + Opus 顾问得分 41.2%（单Haiku 仅 19.7%）
- **Terminal Bench 2.0**: Sonnet + Opus 优于 Sonnet 单兵

## 何时使用

✅ **适合的场景：**
- 复杂编码 Agent 任务
- 计算机使用任务
- 多阶段研究流程
- 需要良好规划的长任务

❌ **不适合的场景：**
- 单回合 Q&A（无计划）
- 用户已自行选择成本和质量的纯路由任务
- 每回合都需要充分发挥顾问能力的工作负载

## 实现

通过 Claude API 的 `advisor_20260301` 工具实现：

```python
response = client.messages.create(
    model="claude-sonnet-4-6",  # 执行者
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",
            "max_uses": 3,  # 可选：限制顾问调用次数
        },
        # ... 其他工具
    ],
    messages=[...]
)
```

## 计费

- 顾问代币按顾问模型的费率计费
- 执行者代币按执行者模型的费率计费
- 顾问通常输出 400-700 文本令牌（包含思考共 1400-1800 令牌）
- 整体成本远低于端到端运行 Opus 模型

## 最佳实践

1. **早期调用**: 在几个探索性读取后尽早调用顾问
2. **结束时调用**: 困难任务在文件写入和测试输出后再次调用
3. **简洁指令**: 添加 `"The advisor should respond in under 100 words"` 可减少约 35-45% 顾问输出
4. **搭配 Effort 设置**: 编码任务用中等 effort 的 Sonnet + Opus 顾问可接近默认 effort 的 Sonnet 水平

## 相关资源

- [[openaicodex|OpenAI Codex]] - 类似的 Agent 架构
- [[LLM Wiki Architecture|LLM Wiki 架构]] - 知识管理架构
