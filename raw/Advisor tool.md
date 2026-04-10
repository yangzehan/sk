---
title: "Advisor tool"
source: "https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool"
author:
published:
created: 2026-04-10
description: "Pair a faster executor model with a higher-intelligence advisor model that provides strategic guidance mid-generation."
tags:
  - "raw"
---
顾问工具允许更快、成本更低的 **执行者模型** 在中期咨询一个高智能 **的顾问模型** ，提供战略指导。顾问阅读完整对话，制定计划或方向修正（通常为400到700个文本标记，包括思考总计1400到1800个标记），执行者继续执行任务。

这种模式适合长期的代理工作量（编码代理、计算机使用、多阶段研究流程），大多数转向是机械的，但拥有完善的计划至关重要。你能接近顾问的个体质量，而大部分代币生成率是执行者模型。

顾问工具还处于测试阶段。请在请求中包含测试版标题。如需申请访问权限或分享反馈，请联系您的Anthropic 客户团队。 `advisor-tool-2026-03-01`

此功能适用于 [零数据保留（ZDR）。](https://platform.claude.com/docs/en/build-with-claude/api-and-data-retention) 当你的组织采用ZDR协议时，通过该功能发送的数据在API响应返回后不会被存储。

## 何时使用它

早期基准显示以下配置有显著提升：

- **你目前在复杂任务中使用 Sonnet：** 将Opus作为顾问，提供同等或更低的总成本的优质提升。
- **你目前使用俳句，想要提升智力：** 添加 Opus 作为顾问。成本会比单一Haiku高，但比换成更大型号的执行器要低。

结果取决于任务。根据自己的工作量评估。

顾问在单回合的问答（无计划）、纯传递式模型选择器（用户已经自行选择成本和质量）或每回合都必须充分发挥顾问模型能力的工作负载中，则不那么适合。

## 模型兼容性

执行者模型（顶层字段）和顾问模型（工具定义内的字段）必须形成有效对。顾问必须至少与执行人一样有能力。 `model` `model`

| 执行者模型 | 顾问模型 |
| --- | --- |
| 克劳德俳句 4.5 （ `claude-haiku-4-5-20251001`) | 克劳德作品4.6（ `claude-opus-4-6`) |
| 克洛德十四行诗 4.6 （ `claude-sonnet-4-6`) | 克劳德作品4.6（ `claude-opus-4-6`) |
| 克劳德作品4.6（ `claude-opus-4-6`) | 克劳德作品4.6（ `claude-opus-4-6`) |

如果你请求无效的组合，API 会返回一个不支持的组合名称。 `400 invalid_request_error`

## 平台可用性

该顾问工具目前在Claude API（Anthropic）上以测试版形式提供。

## 快速入门

```
curl https://api.anthropic.com/v1/messages \
    --header "x-api-key: $ANTHROPIC_API_KEY" \
    --header "anthropic-version: 2023-06-01" \
    --header "anthropic-beta: advisor-tool-2026-03-01" \
    --header "content-type: application/json" \
    --data '{
        "model": "claude-sonnet-4-6",
        "max_tokens": 4096,
        "tools": [
            {
                "type": "advisor_20260301",
                "name": "advisor",
                "model": "claude-opus-4-6"
            }
        ],
        "messages": [{
            "role": "user",
            "content": "Build a concurrent worker pool in Go with graceful shutdown."
        }]
    }'
```

## 工作原理

当你把顾问工具添加到数组时，执行者模型会决定何时调用它，就像其他工具一样。当执行人召唤顾问时： `tools`

1. 执行者发出一个块，且为空。执行者负责计时信号;服务器提供上下文。 `server_tool_use` `name: "advisor"` `input`
2. Anthropic 在顾问模型的服务器端运行一个独立的推理通道，传递执行者的完整转录。顾问会看到系统提示、所有工具定义、所有之前的回合以及所有之前的工具结果。
3. 顾问的回复会以区块的形式返回给执行人。 `advisor_tool_result`
4. 执行人会根据建议继续创作。

所有这些都发生在一个请求中。你这边没有额外的往返。 `/v1/messages`

顾问本身没有工具，也没有上下文管理。它的思维块在结果返回前被放下;只有建议文本能传达到执行人手中。

## 工具参数

| 参数 | 类型 | 默认 | 描述 |
| --- | --- | --- | --- |
| `type` | 弦 | *要求* | 一定是。 `"advisor_20260301"` |
| `name` | 弦 | *要求* | 一定是。 `"advisor"` |
| `model` | 弦 | *要求* | 顾问模型ID，例如。按该模型的子推断费率计费。 `"claude-opus-4-6"` |
| `max_uses` | 整数 | 无限 | 单次请求允许的最大顾问通话次数。一旦执行人达到上限，顾问会再次呼叫并回复，执行人则继续进行，无需进一步建议。这是按请求的上限，而不是每次对话的上限;关于对话层级限制，请参见 [成本控制](#cost-control) 。 `advisor_tool_result_error` `error_code: "max_uses_exceeded"` |
| `caching` | 对象 \|无效 | `null` （关掉） | 支持顾问在通话中快速缓存自己的文字记录。参见 [Advisor提示缓存](#advisor-prompt-caching) 。 |

该物体具有 的形状为 。与内容块不同，这不是断点标记;它是一个开关。服务器决定缓存边界的位置。 `caching` `{"type": "ephemeral", "ttl": "5m" | "1h"}` `cache_control`

## 反应结构

### 成功的顾问接纳

当顾问被调用时，助理内容中会跟随一个区块： `server_tool_use` `advisor_tool_result`

```
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Let me consult the advisor on this."
    },
    {
      "type": "server_tool_use",
      "id": "srvtoolu_abc123",
      "name": "advisor",
      "input": {}
    },
    {
      "type": "advisor_tool_result",
      "tool_use_id": "srvtoolu_abc123",
      "content": {
        "type": "advisor_result",
        "text": "Use a channel-based coordination pattern. The tricky part is draining in-flight work during shutdown: close the input channel first, then wait on a WaitGroup..."
      }
    },
    {
      "type": "text",
      "text": "Here's the implementation. I'm using a channel-based coordination pattern to avoid writer starvation..."
    }
  ]
}
```

总是空的。服务器会自动根据完整转录构建顾问的观点;执行人提交的内容都没有送到顾问手中。 `server_tool_use.input` `input`

### 结果变体

这个领域是一个受歧视的工会。你获得的变体取决于顾问模式： `advisor_tool_result.content`

| 变体 | 场 | 归来时间 |
| --- | --- | --- |
| `advisor_result` | `text` | 顾问模型返回明文（例如，Claude Opus 4.6）。 |
| `advisor_redacted_result` | `encrypted_content` | 顾问模型返回加密输出。 |

当 ，字段包含人类可读的建议。当 时，场中包含一个你无法读取的不透明斑点;下一回合，服务器解密并把明文渲染成执行者的提示符。 `advisor_result` `text` `advisor_redacted_result` `encrypted_content`

在这两种情况下，后续回合都要逐字返回内容。如果你在对话中途切换顾问模式，可以同时处理两种形式。 `content.type`

如果顾问呼叫失败，结果会带有错误：

```
{
  "type": "advisor_tool_result",
  "tool_use_id": "srvtoolu_abc123",
  "content": {
    "type": "advisor_tool_result_error",
    "error_code": "overloaded"
  }
}
```

执行人发现错误后继续执行，未给予进一步建议。请求本身不会失败。

| `error_code` | 含义 |
| --- | --- |
| `max_uses_exceeded` | 请求达到了工具定义设定的上限。同样请求的顾问呼叫后，返回了该错误。 `max_uses` |
| `too_many_requests` | 顾问子推断受费率限制。 |
| `overloaded` | 顾问子推断达到了容量上限。 |
| `prompt_too_long` | 该成绩单超过了顾问模型的上下文窗口。 |
| `execution_time_exceeded` | 顾问的副推断超时了。 |
| `unavailable` | 任何其他导师的失败。 |

顾问费率限制与直接调用顾问模型的调用来自同一模型的值。顾问的费率限制显示在工具结果中;执行程序的速率限制导致整个请求在HTTP 429下失败。 `too_many_requests`

## 多回合对话

在后续回合中，将完整的助理内容，包括方块，传回给API： `advisor_tool_result`

```
import anthropic

client = anthropic.Anthropic()

tools = [
    {
        "type": "advisor_20260301",
        "name": "advisor",
        "model": "claude-opus-4-6",
    }
]

messages = [
    {
        "role": "user",
        "content": "Build a concurrent worker pool in Go with graceful shutdown.",
    }
]

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    betas=["advisor-tool-2026-03-01"],
    tools=tools,
    messages=messages,
)

# Append the full response content, including any advisor_tool_result blocks
messages.append({"role": "assistant", "content": response.content})

# Continue the conversation
messages.append({"role": "user", "content": "Now add a max-in-flight limit of 10."})

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    betas=["advisor-tool-2026-03-01"],
    tools=tools,
    messages=messages,
)
```

如果你在后续回合中省略顾问工具，而消息历史仍包含区块，API 会返回一个 。 `tools` `advisor_tool_result` `400 invalid_request_error`

顾问工具没有内置的对话层级上限。限制顾问 通话中，按客户端计算。当你达到你的 天花板，从你的数组中移除顾问工具 **，并** 从消息历史中移除所有方块，以避免出现 。 `tools` `advisor_tool_result` `400 invalid_request_error`

## 流媒体

顾问子推断不流。执行者的流在顾问运行时暂停，然后完整结果在单一事件中出现。

带有顾问来电开始的信号区块。暂停从该块关闭（）开始。暂停期间，流中除大约每30秒发出的标准SSE保持活外，流路较为安静;短期顾问通话可能没有延迟。 `server_tool_use` `name: "advisor"` `content_block_stop` `ping`

当顾问完成后，他们会在一个事件中完全成型（没有变化）。执行者输出随后恢复流式传输。 `advisor_tool_result` `content_block_start`

随后会触发一个事件，更新后的数组反映顾问的令牌计数。 `message_delta` `usage.iterations`

## 使用与计费

顾问调用作为一个单独的子推断，按顾问模型的费率计费。使用情况在数组中报告： `usage.iterations[]`

```
{
  "usage": {
    "input_tokens": 412,
    "cache_read_input_tokens": 0,
    "cache_creation_input_tokens": 0,
    "output_tokens": 531,
    "iterations": [
      {
        "type": "message",
        "input_tokens": 412,
        "cache_read_input_tokens": 0,
        "cache_creation_input_tokens": 0,
        "output_tokens": 89
      },
      {
        "type": "advisor_message",
        "model": "claude-opus-4-6",
        "input_tokens": 823,
        "cache_read_input_tokens": 0,
        "cache_creation_input_tokens": 0,
        "output_tokens": 1612
      },
      {
        "type": "message",
        "input_tokens": 1348,
        "cache_read_input_tokens": 412,
        "cache_creation_input_tokens": 0,
        "output_tokens": 442
      }
    ]
  }
}
```

顶层字段仅反映执行者代币。顾问代币不会被计入顶层总额，因为它们的计费率不同。与 的迭代按顾问模型的费率计费;与 的迭代按执行者模型的费率计费。 `usage` `type: "advisor_message"` `type: "message"`

聚合规则因字段而异。顶层是所有执行者迭代的总和。顶层，仅反映第一个执行者迭代;后续执行者迭代的输入不会被重叠，因为它们包含了之前的输出令牌。用于构建成本跟踪逻辑时的完整每次迭代分解。 `output_tokens` `input_tokens` `cache_read_input_tokens` `usage.iterations`

顾问的输出通常为400到700个文本代币，或包括思考在内的总共1400到1800个代币。节省的成本来自于顾问没有产出全部最终成果;执行人则以较低的费率完成此交易。

顶层仅适用于执行者输出。它不绑定顾问子推理代币。顾问的代币也不从执行人获得的任何任务预算中提取。 `max_tokens`

## 顾问提示缓存

缓存有两个独立层。

### 执行端缓存

该块像其他内容块一样可缓存。在后续回合放置的断点会触发。执行者的提示总是包含明文建议，无论客户端是否收到 或 ，因此缓存行为在两种结果变体中是相同的。 `advisor_tool_result` `cache_control` `text` `encrypted_content`

### 顾问端缓存

启用工具定义，以实现同一次通话中导师自身的文字记录的快速缓存： `caching`

```
tools = [
    {
        "type": "advisor_20260301",
        "name": "advisor",
        "model": "claude-opus-4-6",
        "caching": {"type": "ephemeral", "ttl": "5m"},
    }
]
```

第N次通话的顾问提示词是第N次通话的提示，加上一个段，因此前缀在各通话间是稳定的。启用后，每个顾问调用都会写入缓存条目;下一通电话会读到这个点，只支付Delta的费用。你会看到在第二轮及以后迭代中变为非零。 `caching` `cache_read_input_tokens` `advisor_message`

**何时启用：** 缓存写入的成本高于读取，除非每次对话调用顾问两次或更少。缓存大约在三次顾问电话时收支平衡，之后会有所改善。启用它以实现长代理循环;短时间任务时保持关闭。

**保持一致：** 设置一次，整个对话都保持不动。在对话中切换开关会触发缓存未命中。 `caching`

[`clear_thinking`](https://platform.claude.com/docs/en/build-with-claude/context-editing) 当值非偏移时，每回合会变更导师引用的成绩单， 导致顾问端缓存未命中。这只是成本下降;建议 质量不受影响。当未明确配置的情况下启用扩展思维时，API 默认为 ，从而触发此行为。 设置为保持顾问缓存的稳定性。 `keep` `"all"` `clear_thinking` `keep: {type: "thinking_turns", value: 1}` `keep: "all"`

## Combining with other tools

The advisor tool composes with other server-side and client-side tools. Add them all to the same array:`tools`

```
tools = [
    {
        "type": "web_search_20250305",
        "name": "web_search",
        "max_uses": 5,
    },
    {
        "type": "advisor_20260301",
        "name": "advisor",
        "model": "claude-opus-4-6",
    },
    {
        "name": "run_bash",
        "description": "Run a bash command",
        "input_schema": {
            "type": "object",
            "properties": {"command": {"type": "string"}},
        },
    },
]
```

The executor can search the web, call the advisor, and use your custom tools in the same turn. The advisor's plan can inform which tools the executor reaches for next.

| Feature | Interaction |
| --- | --- |
| [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) | Supported. is reported per item.`usage.iterations` |
| [Token counting](https://platform.claude.com/docs/en/build-with-claude/token-counting) | Returns the executor's first-iteration input tokens only. For a rough advisor estimate, call with set to the advisor model and the same messages.`count_tokens` `model` |
| [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing) | `clear_tool_uses` is not yet fully compatible with advisor tool blocks; full support is planned for a follow-up release. With, see the caching warning above.`clear_thinking` |
| `pause_turn` | A dangling advisor call ends the response with and the block as the last content block. The advisor executes on resumption. See [Server tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/server-tools#the-server-side-loop-and-pause-turn).`stop_reason: "pause_turn"` `server_tool_use` |

## Best practices

### Prompting for coding and agent tasks

The advisor tool ships with a built-in description that nudges the executor to call it near the start of complex tasks and when it hits difficulty. For research tasks, no additional prompting is typically needed.

On coding and agent tasks, the advisor produces higher intelligence at similar cost when it reduces total tool calls and conversation length. Two timings drive this improvement:

1. An early first advisor call, after a few exploratory reads are in the transcript.
2. For difficult tasks, a final advisor call after file writes and test outputs are in the transcript.

If your agent exposes other planner-like tools (for example, a todo list tool), prompt the model to call the advisor before those tools so the advisor's plan funnels into them. The suggested system prompt below reinforces the early-call pattern; add your own funnel-in sentence pointing at whichever planner tools your agent exposes.

#### Suggested system prompt for coding tasks

For coding tasks where you want consistent advisor timing and around two to three calls per task, prepend the following blocks to your executor system prompt before any other sentences that mention the advisor. On internal coding evaluations this pattern produced the highest intelligence at near-Sonnet cost.

Timing guidance:

```
You have access to an \`advisor\` tool backed by a stronger reviewer model. It takes NO parameters — when you call advisor(), your entire conversation history is automatically forwarded. They see the task, every tool call you've made, every result you've seen.

Call advisor BEFORE substantive work — before writing, before committing to an interpretation, before building on an assumption. If the task requires orientation first (finding files, fetching a source, seeing what's there), do that, then call advisor. Orientation is not substantive work. Writing, editing, and declaring an answer are.

Also call advisor:
- When you believe the task is complete. BEFORE this call, make your deliverable durable: write the file, save the result, commit the change. The advisor call takes time; if the session ends during it, a durable result persists and an unwritten one doesn't.
- When stuck — errors recurring, approach not converging, results that don't fit.
- When considering a change of approach.

On tasks longer than a few steps, call advisor at least once before committing to an approach and once before declaring done. On short reactive tasks where the next action is dictated by tool output you just read, you don't need to keep calling — the advisor adds most of its value on the first call, before the approach crystallizes.
```

How the executor should treat the advice (place directly after the timing block):

```
Give the advice serious weight. If you follow a step and it fails empirically, or you have primary-source evidence that contradicts a specific claim (the file says X, the paper states Y), adapt. A passing self-test is not evidence the advice is wrong — it's evidence your test doesn't check what the advice is checking.

If you've already retrieved data pointing one way and the advisor points another: don't silently switch. Surface the conflict in one more advisor call — "I found X, you suggest Y, which constraint breaks the tie?" The advisor saw your evidence but may have underweighted it; a reconcile call is cheaper than committing to the wrong branch.
```

#### Trimming advisor output length

Advisor output is the advisor's largest cost driver. To reduce that cost, prepend a single conciseness instruction to the system prompt before any other sentence that mentions the advisor. In internal testing, the following line cut total advisor output tokens by roughly 35 to 45 percent without changing call frequency:

```
The advisor should respond in under 100 words and use enumerated steps, not explanations.
```

Pair this with the timing block above for the strongest cost-versus-quality tradeoff.

### Pairing with effort settings

For coding tasks, pairing a Sonnet executor at medium [effort](https://platform.claude.com/docs/en/build-with-claude/effort) with an Opus advisor achieves intelligence comparable to Sonnet at default effort, at lower cost. For maximum intelligence, keep the executor at default effort.

### Cost control

- For conversation-level budgets, count advisor calls client-side. When you reach your cap, remove the advisor tool from **and** strip all blocks from your message history to avoid a.`tools` `advisor_tool_result` `400 invalid_request_error`
- Enable only for conversations where you expect three or more advisor calls.`caching`

## Limitations

- **Advisor output does not stream.** Expect a pause in the stream while the sub-inference runs.
- **No built-in conversation-level cap on advisor calls.** Track and cap them client-side.
- **`max_tokens` applies to executor output only.** It does not bound advisor tokens.
- **Anthropic Priority Tier** is honored per model. Priority Tier on the executor model does not extend to the advisor; you need Priority Tier on the advisor model specifically.

Was this page helpful?