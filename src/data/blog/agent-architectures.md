---
author: Chiho Song
pubDatetime: 2026-11-03T13:45:00Z
title: Agent Architectures for Applied AI Engineers
slug: agent-architectures
featured: false
draft: false
tags:
  - applied-ai-engineer
  - ai-agent
  - llm
  - interview
description: Workflows vs. agents, the agent loop, common patterns, multi-agent trade-offs, and how to keep agents controllable, debuggable, and affordable.
lang: en
---

"Design an agent that does X" is now a standard Applied AI interview prompt. The weak answer jumps straight to a fully autonomous multi-agent system. The strong answer starts with the simplest design that works, explains when more autonomy is worth the cost, and shows how to keep it under control.

This post builds on the [tool calling](/posts/tool-calling/) post. Tool calling is one step; an agent is what happens when you run that step in a loop.

---

## Workflows vs. Agents

Anthropic's "Building effective agents" draws a useful line:

- **Workflows**: LLMs and tools orchestrated through **predefined code paths.** Your code decides the steps.
- **Agents**: The **LLM dynamically directs** its own process and tool use. The model decides the steps.

| Choose a workflow when…                | Choose an agent when…                                |
| -------------------------------------- | ---------------------------------------------------- |
| The steps are known in advance         | The number and order of steps can't be predicted     |
| You need predictable latency and cost  | Flexibility matters more than predictability         |
| Errors must be easy to trace           | The task is open-ended (research, coding, debugging) |
| A single LLM call plus retrieval works | Success can be checked (tests pass, ticket resolved) |

The key interview point: **autonomy is a cost, not a feature.** Agents trade latency, cost, and predictability for flexibility. Start with a single LLM call, add a workflow if needed, and use an agent only when the task really requires model-directed control.

---

## The Agent Loop

At its core, an agent is a loop:

```text
goal + context + tools
        ↓
  ┌─→ model decides next action
  │         ↓
  │   tool call → execute → result
  │         ↓
  └── add result to context
        ↓ (stop condition met)
     final answer
```

**Stopping conditions** matter as much as the loop:

- The model says it's done (no more tool calls).
- A verifiable goal is reached (tests pass, the file exists, the API returns success).
- A budget is hit: maximum steps, tokens, time, or money.
- A checkpoint requires human approval.

An agent without hard budgets will eventually loop on a confusing error until it burns through your API bill.

---

## Common Workflow Patterns

Most production "agents" are actually one of these workflows, and that's fine.

1. **Prompt chaining**: Break a task into fixed steps, with each output feeding the next. Add checks between steps. Example: outline → draft → edit.
2. **Routing**: Classify the input, then send it to a specialized prompt, tool set, or model. Example: send billing questions and technical questions to different handlers; send easy queries to a cheaper model.
3. **Parallelization**: Run independent subtasks at the same time (sectioning), or run the same task several times and vote. Example: check a document against five policies in parallel.
4. **Orchestrator-workers**: A central LLM breaks a task into subtasks it decides at runtime, delegates them to workers, and combines the results. Example: code changes across an unknown number of files.
5. **Evaluator-optimizer**: One LLM generates, another evaluates and gives feedback, and they iterate. Example: translation or writing with clear quality criteria.

Knowing these patterns lets you answer design questions with "this is routing plus parallelization" instead of "an agent figures it out."

---

## ReAct and Planning

**ReAct** (Yao et al., 2022) interleaves **reasoning** and **acting**: the model writes a thought, takes an action, observes the result, and reasons again. Reasoning helps it decide what to do; observations ground the reasoning in real data instead of guesses. Modern tool-calling agents follow this shape, often with the model's reasoning handled internally.

**Plan-then-execute** has the model write an explicit plan first, then work through it, revising when steps fail. Plans make long tasks more stable and easier to inspect, and they're a natural checkpoint for human review.

---

## Memory and State

Agents run into context limits quickly: every tool call and result stays in the conversation.

- **Short-term (context window)**: The current conversation, tool results, and scratchpad notes.
- **Compaction**: Summarize or drop old tool results when the context gets long.
- **External memory**: Write notes, plans, and progress to files or a database, and read them back when needed. This also lets a task survive across sessions.
- **Retrieval**: Pull relevant past information instead of keeping everything in context.

Good agents treat context as a scarce resource: keep what's needed for the next decision and store the rest outside.

---

## Multi-Agent Systems

A multi-agent system usually means a **lead agent** that spawns **sub-agents**, each with its own context, to work on parts of a task.

**When it helps**

- **Parallel exploration**: Several independent research directions at once.
- **Context isolation**: Each sub-agent explores in its own context and returns a condensed result, keeping the lead agent's context clean.
- **Breadth-first tasks**: Problems that split naturally into independent pieces.

Anthropic reported that its multi-agent research system (a Claude Opus 4 lead with Claude Sonnet 4 sub-agents) outperformed a single Claude Opus 4 agent by 90.2% on an internal research eval.

**When it hurts**

- **Cost**: In the same write-up, Anthropic noted agents use about 4× the tokens of chat, and multi-agent systems about 15×. The task has to be valuable enough to pay for that.
- **Coordination**: Sub-agents can duplicate work, miss context the lead had, or make conflicting decisions.
- **Tightly coupled tasks**: Most coding tasks, where every step depends on shared state, don't split well.

Interview framing: multi-agent is a tool for **parallelizable, high-value, breadth-heavy** tasks, not a default architecture.

---

## Control and Safety

- **Budgets**: Cap steps, tokens, wall-clock time, and spend per task.
- **Permissions**: Give tools least privilege; enforce authorization in code.
- **Human-in-the-loop**: Require approval before irreversible or costly actions, and at plan checkpoints for long tasks.
- **Sandboxing**: Run code and file operations in isolated environments.
- **Untrusted inputs**: Treat web pages, emails, and documents the agent reads as data, not instructions (prompt injection).
- **Escalation paths**: When stuck, the agent should ask for help or hand off, not keep trying forever.

---

## Evaluating and Debugging Agents

Agents fail in long, branching ways, so you need visibility.

- **Traces**: Log every model call, tool call, argument, result, and token count, linked by task ID.
- **Outcome evals**: Did the task succeed in a sandbox with a checkable end state?
- **Trajectory evals**: Were the tool choices and arguments sensible? Were there wasted or risky steps?
- **Repeat runs**: Run each task several times; agents are non-deterministic, and a 20% failure rate only shows up across runs.
- **Replay**: Re-run a failed trace from the failing step with a changed prompt or tool to test a fix.

The [LLM evals](/posts/llm-evals/) post covers these methods in more depth.

---

## Interview Questions

1. **What's the difference between a workflow and an agent?**
   A workflow follows code-defined steps; an agent lets the model choose its steps and tools dynamically. Use workflows for predictable tasks and agents for open-ended ones where steps can't be known in advance and success can be verified.

2. **Walk through the agent loop. How does it stop?**
   The model picks an action, the app executes the tool, the result is added to context, and the loop repeats. It stops when the model finishes, a verifiable goal is met, a budget runs out, or a human checkpoint is reached.

3. **Describe ReAct.**
   The model alternates reasoning and actions: think, act, observe, repeat. Reasoning guides tool use, and observations ground the reasoning in real results.

4. **When does a multi-agent design beat a single agent?**
   For parallelizable, breadth-heavy tasks where isolated sub-agent contexts help and the value justifies much higher token cost. For tightly coupled tasks, a single agent is usually better.

5. **How do you stop an agent from running away?**
   Hard limits on steps, tokens, time, and spend; least-privilege tools; human approval for irreversible actions; sandboxing; and escalation when stuck.

6. **How do you debug an agent that fails 20% of the time?**
   Collect traces from many runs, group failures by where they happen, check whether it's tool selection, bad arguments, missing context, or ambiguous tool results, fix the most common cause, and confirm with repeated-run evals.

---

## References

- [Anthropic, Building effective agents (2024)](https://www.anthropic.com/engineering/building-effective-agents)
- [Yao et al., ReAct: Synergizing Reasoning and Acting in Language Models (2022)](https://arxiv.org/abs/2210.03629)
- [Anthropic, How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
