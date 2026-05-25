---
title: "The advisor strategy: Give Sonnet an intelligence boost with Opus"
source: "https://claude.com/blog/the-advisor-strategy"
author:
published: 2001-04-09
created: 2026-04-10
description: "Pair Opus as an advisor with Sonnet or Haiku as an executor, and get Opus-level intelligence in your agents at a fraction of the cost."
tags:
  - "raw"
---
想要更好地平衡智能与成本的开发者，已经趋向于我们所说的顾问策略：让Opus担任顾问，Sonnet或Haiku作为执行者。这为你的代理人带来接近Opus级别的智能，同时成本保持在十四行诗水平。

今天我们将在Claude平台上推出顾问工具，使顾问策略只需在API调用中进行一行更改即可。

## 通过顾问策略打造具成本效益的经纪人

![[aaf94682ae755fe78bd1289f8ee45b40_MD5.jpg]]

在顾问策略中，Sonnet或Haiku作为执行者，端到端运行任务，调用工具，读取结果，并迭代寻找解决方案。当执行人做出无法合理解决的决定时，会咨询Opus作为顾问的指导。Opus 访问共享上下文并返回计划、修正或停止信号，执行者继续执行。顾问从不调用工具，也不生成面向用户的输出，只为执行者提供指导。

这颠覆了常见的子代理模式，即较大的编排器模型分解工作并委派给较小的工作者模型。在顾问战略中，一个更小、更具成本效益的模型驱动并升级，无需分解、员工池或编排逻辑。边境层级推理仅在执行人需要时适用，其余运行仍维持执行人级别成本。

在我们的评估中，Sonnet与Opus作为顾问合作，显示在 [SWE-bench 多语言](https://www.swebench.com/multilingual.html) <sup>1</sup> 仅相较于十四行诗，同时将每项能动任务的成本降低了11.9%。

![[e94d979a1c47c79db35a8c8bfd6006a1_MD5.jpg]]

## 顾问工具

我们将顾问策略引入我们的API，并用 [**顾问工具**](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) ，这是一个服务器端工具，Sonnet和Haiku知道在需要指导或帮助完成特定任务时调用它。

在我们的评估中，Sonnet与Opus顾问合作后，BrowseComp的成绩有所提升 <sup>2</sup> 以及终端工作台2.0 <sup>3</sup> 而每项任务的成本却低于单纯Sonnet。

![[b03b1be98c31df40c6e94a5668a81898_MD5.jpg]]

顾问策略也与Haiku作为执行人合作。在BrowseComp上，Haiku与Opus顾问合作得分为41.2%，是单人19.7%的两倍多。《俳句》与《Opus》导师合作，单人得分落后《十四行诗》29%，但每项任务成本低85%。导师相对于单纯俳句会增加成本，但综合价格仍是十四行诗的一小部分，因此对于需要智力与成本平衡的高量任务来说，它是强有力的选择。

![[166a1305c44c29e743a0f8b58ddcced0_MD5.jpg]]

  
在你的 Messages API 请求中声明advisor\_20260301，模型交接将在单一的 /v1/messages 请求内完成——无需额外的往返或上下文管理。执行人模型决定何时调用该协议。当它出现时，我们会将策划的上下文路由到顾问模型，返回计划，执行者继续在同一请求内。  

```python
response = client.messages.create(
    model="claude-sonnet-4-6",  # executor
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",
            "max_uses": 3,
        },
        # ... your other tools
    ],
    messages=[...]
)

# Advisor tokens reported separately
# in the usage block.
```

**价格** 。顾问代币按顾问模型的费率计费;执行者代币按执行者模型的费率计费。由于顾问只生成短期计划（通常为400-700个文本代币），而执行者则以较低速率处理全部输出，整体成本远低于端到端运行顾问模型。**  
  
内置成本控制。** 设置max\_uses限制每次请求的顾问通话次数。顾问代币会单独在使用区块报告，这样你可以追踪每个等级的支出。

**可以和你现有的工具一起工作。** 顾问工具只是你 Messages API 请求中的另一个条目。你的经纪人可以 [在网上搜索](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool),[执行代码](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool) ，并在同一循环中查阅Opus。

0/5

## 开始

该顾问工具现已在Claude平台上原生提供测试版。开始：

1. 添加测试版功能标题：anthropic-beta： advisor-tool-2026-03-01
2. 将advisor\_20260301添加到你的Messages API请求中
3. 修改你的 [系统提示](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool#suggested-system-prompt-for-coding-tasks) 根据你的使用场景

我们建议你用Sonnet sola、Sonnet executor with Opus advisor和Opus solo来运行现有的评估套件。探索 [纪录片](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) 了解更多。

## 注释

1. **软件工作台多语言者：** 十四行诗4.6独唱采用了适应性思维。Sonnet 4.6 + Advisor 在关闭思考的情况下使用了我们推荐的编码系统提示。两次运行都大量使用了bash和文件编辑工具。分数通过五次试验平均，涵盖九种语言、共300道题。Opus 4.6在所有运行中都被用作顾问模型。
2. **浏览电脑：** 所有运行都关闭了网页搜索和网页取材工具。十四行诗4.6的创作使用了中等努力。Sonnet 4.6 + Advisor 使用了我们推荐的系统提示来编码;俳句4.5+顾问则没有。没有程序化工具调用或上下文压缩。评分基于1,266道题，每题一次尝试。Opus 4.6在所有运行中都被用作顾问模型。
3. **终端工作台2.0：** 所有运行都用关闭了bash和文件编辑工具的想法。十四行诗4.6的创作使用了中等努力。两次导师的运行都没有使用我们推荐的系统提示来编码。每个任务在一个隔离舱中运行，资源分配为3倍，超时时间为1倍。得分平均为每项任务的5次尝试，涵盖89个任务。Opus 4.6在所有运行中都被用作顾问模型。

## 用Claude彻底改变你的组织运作方式

订阅开发者通讯

产品更新、操作指南、社区聚焦灯光，以及更多内容。每月发送至您的邮箱。