---
title: "Building Effective Human-Agent Teams"
source: "https://claude.com/blog/building-effective-human-agent-teams"
author: "Kristen Swanson (Anthropic Education team)"
published: 2026-06
retrieved: 2026-06-25
description: "Anthropic introduces Claude Tag and shares lessons learned for building multiplayer agents — AI that works alongside many humans in shared workspaces like Slack."
tags:
  - "clippings"
  - "claude"
  - "agent"
  - "human-agent-collaboration"
related-products:
  - "Claude Tag"
  - "Agent Teams (Claude Code)"
---

# Building Effective Human-Agent Teams

> **Author**: Kristen Swanson (Education team, Anthropic)
> **Acknowledgements**: Matt Bell, Erik Olesund, Hasnain Lakhani, Shale Craig, Nolan Caudill, Mike Schiraldi, Aleks Todorova, Molly Vorwerck
> **Source**: <https://claude.com/blog/building-effective-human-agent-teams>

---

## 引言

Working with AI used to mean one person interfacing with a single chat window. Over time, AI has become increasingly capable at handling complex, long-running work, like coding, research, and financial analysis. With this, we've seen many new ways to use AI — from the terminal and IDE to spreadsheets and decks — but the work has still very much been a "single-player" experience: one human worked with one agent to accomplish individual tasks.

This is changing with the release of tools like **Claude Tag**. Now, humans and agents can work together in the same workspace, collaborating in service of goals shared by a team. Work now looks a lot more like a *multiplayer game*, with teams of humans setting the strategy, and Claude executing the work.

This involves some new ways of working. At Anthropic, we've been testing the technology required to make human-agent teams successful for the last several months. In this article, we explain what multiplayer agents are, and the lessons we've learned for building with them.

---

## 什么是 Multiplayer Agents？

"Multiplayer agents" is how we refer here to AI models that work with many different humans at the same time. Much like regular agents, they have their own memory and skills. But in other respects they're quite different. They have their own credentials and they live in places where work happens. At Anthropic, that's inside team collaboration tools like Slack.

For agents to productively participate in a team channel, they need specific capabilities:

- **Persistent memory** — so they can remember goals and tune their execution towards them
- **Credentials not tied to humans** — so they can operate within safe, predictable guardrails
- **Ongoing broad access to information** — so they can learn how the organization works and take action to execute tasks in service of the team's goals

These capabilities amount to the technical foundation required for an agent to participate productively across a team of many humans. However, making human-agent teams *successful* requires more than this: teams need specific ways of working and shared norms, too.

---

## Lesson 1: Work in public and give agents broad context

Teams at Anthropic share information proactively and openly. This is especially true when agents are on the team, because agents build their understanding entirely from the text a team makes searchable: Slack, code, docs, and meeting notes. Private messages, hallway conversations, and restricted documents can't provide agents with context. **For an agent, if it's not written down and accessible, it doesn't exist.**

Instead of deciding what information should be available to agents one doc or Slack channel at a time, we use clearly defined security boundaries that apply to entire Slack workspaces, as well as to meeting transcripts and doc libraries. Within the security boundary, context flows to every teammate — whether human or AI.

A high degree of transparency has a reward:

- Agents that can read decisions from team meetings won't suggest tasks or projects that were deprioritized
- Agents with access to product specs beyond their own team can recommend patterns that have succeeded for others
- Agents can read enormous volumes of text far faster than humans do, so they routinely surface relevant work that humans would otherwise have missed

At Anthropic, working in public looks like:

- Choosing a handful of security boundaries at the company and creating workspaces and document sharing settings that match each security boundary
- Defaulting new communication channels to public within the organization, and ensuring decisions land in channels, docs, and meeting notes every time
- Writing artifacts and meeting notes so that agents can find them, since agents are now a primary consumer of team documentation
- Making sure AI has access to the right tools and information needed to get their job done

> 当然，一些敏感互动仍需要个人与 AI 私下进行。对于这些场景，使用 Claude Tag 可以直接 @Claude 发私聊，或者使用现有的 claude.ai 和 Claude Cowork 应用。

---

## Lesson 2: Every human and agent get a defined role with the right tools for the job

Human-agent teams share one roster, one set of artifacts, and one working space. Agents have their own credentials, skills, and tool access. Different agents also hold different roles: for instance, while one might own the data analysis for a project, another will hold and enforce the design standard, and a third will run research synthesis.

When a project kicks off, humans chat with the agents to figure out which roles to assign, and how the humans and agents will work together.

Once the jobs for humans and agents are clear, an agent might spin up other agents to make sure that specific tasks are handled by the agents with the right memory and appropriate access. Importantly, they need access to all the tools required to accomplish the job: one that handles data analysis might need access to BigQuery, and one that performs QA might need access to the Playwright MCP.

> **Without clear roles, people end up running fleets of personal AIs on the side, duplicating work and fracturing the team's context.**

An engineering team at Anthropic started creating rosters to help codify human and agent roles because it made driving their work much easier and more concrete. Some things that clicked for them early on:

- Specific roles also help humans easily track where responsibility for a task lies, whether that's in individual tasks or an entire team's set of responsibilities
- Writing **skill files** to define specific agents' roles helps to make specialization easy, and allows people across the company to quickly stand up other agents of the same type
- The team adds new agents to focus on new areas when projects get more complex. For example, they added a release manager agent to deal with new software releases.

These methods let humans' mental model of a human-agent team scale as the number of agents grows.

---

## Lesson 3: Set a north star to make agents more proactive

Although some agents at Anthropic simply complete assigned tasks, the most important ones proactively suggest new projects and workstreams. This often happens when a team that has already given its agents rich context and clear roles adds another guide: **a north star**.

North stars are ambitious, wide-reaching goals that help teams decide which tasks and workstreams are the right ones. At Anthropic, humans always set the north star, grounding it in the mission and goals of the business.

Once a north star is clearly articulated in writing, humans share it with the agents on their team. Then, importantly, humans choose which agents should proactively suggest new workstreams to help achieve this long-term goal.

> **Example**: An internal tools team with a north star to "make product onboarding more helpful" saw an agent proactively recommended copy revisions to the onboarding flow error messages. These changes measurably increased onboarding success the following week.

At Anthropic, setting a north star looks like:

- Having humans discuss, debate, and document an ambitious north star goal for their human-agent team — one that's rooted in the company's mission and business goals
- Sharing the north star with agents on the team and explicitly naming which agents can proactively recommend new workstreams
- Keeping high-fidelity human time protected on the calendar, with meetings now focused on the most important work

---

## Lesson 4: Build trust over time

Teams at Anthropic grant agents autonomy in proportion to demonstrated reliability, then expand it deliberately. Engineers have successfully dispatched agents on their team to handle **500 bug fixes independently**, but things certainly didn't start off that way.

When a new human colleague joins the team, it takes time to assess their capabilities and develop strong working routines. The same is true for agents. Users have to experiment with giving agents many different tasks so they can learn what the agent is capable of, how to clearly describe the goal, what skill files it needs, and what prompts work best to elicit a desired behavior.

Notably, we've found that the best long-running agents have many different ways to verify their work before a human looks at it:

- Code has tests, of course
- Technical docs can have rubrics and style guides applied to them
- As with humans, it often helps to give one agent the job of doing the task and another agent the job of checking the first agent's work. This is often called the **"Doer-Verifier" agent harness**

### Engineering leader case study: backlog triage

A new engineering leader inherited a team with a big backlog. To get a handle on it:

1. **Setup**: Invited a few humans and a few agents to sort through the backlog and prioritize what was most important.
2. **Agent teams**:
   - **Triage agents**: Read through all items, figured out if anyone was working on them, assigned complexity scores to anything unowned
   - **Execution agents**: Read from the list, filtered to medium and low complexity items, and created code changes
3. **Early stage**: Humans reviewed every decision made by an agent and marked any that required human input
4. **Mid stage**: Humans taught the agents to surface those decisions to humans directly, ensuring that decisions with hard tradeoffs always had a human in the loop
5. **Weekly report**: Compiled a "lessons & missteps" report so agents kept track of mistakes and avoided making them again
6. **Long-term**: The leader gave more and more complex code changes to agents and spent less time guiding day-to-day tasks

### Treating human attention as scarce

Once agents were more independent, the leader coached them to treat human attention as the scarce resource it is:

- **Batch questions** to be answered in a single pass
- **Repeat key context** to get a human up to speed quickly
- **Limit how many things** each human sees at once

Some teams have agents whose sole role is deciding how to batch and elevate only the most important communication for human team members. Others set guardrails around how much work agents should do per day, so that humans are able to meaningfully engage with the work.

---

## Questions to ask

As you're laying the foundation for your human-agent teams, consider the following questions:

1. Is all the information and access that agents and humans need both public and broadly searchable?
2. Can you write down your team's roster (humans and agents), and say what each member owns?
3. Does every human and agent on the team have access to the right tools to perform their job?
4. Do you have rubrics or tests for humans and agents to verify key work products?
5. Does your team have a clear north star that everyone can reference?

---

## Moving forward

None of these patterns are new — at least not for humans. A strong north star, clear roles, strong documentation, a shared bar for quality, and room to learn from mistakes are the healthy team habits we've known for decades. Agents just make it even more important not to skip them.

The teams getting the most from their agents are the ones who are most intentional about applying these fundamentals.

---

## Links

- **Claude Tag**: <https://www.anthropic.com/news/introducing-claude-tag>
- **Claude Tag support docs**: <https://support.claude.com/en/articles/15594475-what-is-claude-tag>
- **Agent Teams (Claude Code)**: <https://code.claude.com/docs/en/agent-teams>
- **Managed agents memory**: <https://platform.claude.com/docs/en/managed-agents/memory>
- **Managed agents engineering**: <https://www.anthropic.com/engineering/managed-agents>
- **Agent skills overview**: <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview>
- **Context engineering for agents**: <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
- **Equipping agents with skills**: <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
- **Containment**: <https://www.anthropic.com/engineering/how-we-contain-claude>
- **Harness design for long-running apps**: <https://www.anthropic.com/engineering/harness-design-long-running-apps>