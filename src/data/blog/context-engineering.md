---
author: Chiho Song
pubDatetime: 2026-11-03T19:45:00Z
title: Context Engineering for Applied AI Engineers
slug: context-engineering
featured: false
draft: false
tags:
  - applied-ai-engineer
  - context-engineering
  - llm
  - ai-agent
  - interview
description: Deciding what the model sees at each step. Why more context can hurt, how to write system prompts, and how to keep long-running agents from degrading.
lang: en
---

As models get stronger, more production failures come from what's in the context window, or missing from it, than from the model itself. The model had the wrong documents, too many tool results, a vague system prompt, or the one important fact buried in the middle of 100,000 tokens.

**Context engineering** is the discipline of deciding what the model sees at each step. Interviewers probe it with questions like "Why does the agent work for ten steps and then degrade?" or "How would you fit a large codebase into a prompt?"

---

## From Prompt Engineering to Context Engineering

Prompt engineering focuses on the instruction: how to phrase the request.

Context engineering covers **everything the model receives on each call**:

- System prompt and instructions
- Examples
- Retrieved documents
- Tool definitions
- Tool results
- Conversation history
- Memory and notes from earlier work

In a single chat turn, the instruction is most of the context. In an agent running for fifty steps, the instruction is a small fraction; tool results and history dominate. That's why the problem shifts from "write a good prompt" to "manage what accumulates." The [agent architectures](/posts/agent-architectures/) post covers the loop that produces this accumulation.

---

## Context Is a Finite Resource

More context isn't free, and it isn't always better.

- **Cost and latency**: Every input token is billed and processed on every call.
- **Hard limits**: Context windows are large but finite, and long agent runs fill them.
- **Quality degradation**: Models don't use long context uniformly.
  - **Lost in the middle**: Liu et al. (2023) found models used information at the start and end of long inputs much better than information in the middle.
  - **Context rot**: Chroma's 2025 report tested 18 models, holding task difficulty fixed while increasing input length, and found performance became increasingly unreliable as inputs grew, even on simple tasks.

Anthropic's guidance on context engineering frames the goal as finding **the smallest set of high-signal tokens** that produces the outcome you want. Every token you add should earn its place.

---

## Writing Effective System Prompts

The system prompt is the stable part of the context. Anthropic describes aiming for the **right altitude**:

- **Too low**: Brittle, hardcoded if-else rules for every case. They break on anything unanticipated and are hard to maintain.
- **Too high**: Vague guidance ("be helpful and accurate") that assumes shared context the model doesn't have.
- **Right altitude**: Specific enough to guide behavior, flexible enough to give the model heuristics it can apply to new cases.

Practical habits:

- **Structure it** with clear sections: role, task, constraints, tools, output format.
- **Explain why**, not just what. "Keep answers short because users read on mobile" generalizes better than "max 50 words."
- **Use a few diverse examples** instead of a long list of edge-case rules.
- **Start minimal**, test on real cases, and add instructions only to fix observed failures.

---

## Retrieval and Just-in-Time Context

Two strategies for getting information into context:

- **Up front**: Retrieve likely-relevant data before the call and put it in the prompt. Classic [RAG](/posts/rag-fundamentals/). Fast and predictable, but you have to guess what's needed.
- **Just in time**: Give the model lightweight references (file paths, document IDs, search tools) and let it load details when it needs them. This is how coding agents explore a codebase: list files, search, then open only what matters.

Most systems combine them: pre-load a small amount of critical context, and give the agent tools to fetch the rest.

The same applies to tools. Large tool catalogs and verbose tool results are context too. Return compact results, paginate, and load tool definitions on demand when there are many (see [tool calling](/posts/tool-calling/)).

---

## Long-Running Tasks

When a task outgrows one context window, three techniques help:

1. **Compaction**: When the context gets long, summarize the history (decisions, open problems, key facts) and continue from the summary. Clearing old raw tool results is the simplest version: the model rarely needs a file listing from forty steps ago.
2. **Structured note-taking**: The agent writes progress, plans, and findings to external files (a `NOTES.md`, a to-do list) and reads them back later. Memory lives outside the window and survives compaction or a new session.
3. **Sub-agents with isolated context**: Delegate a focused subtask to a sub-agent that explores in its own context and returns a condensed result. The lead agent's context stays clean.

The common idea: **keep in context what the next decision needs, and store everything else somewhere retrievable.**

---

## Ordering and Caching

Order affects both quality and cost.

- **Stable content first**: System prompt, tool definitions, and static reference documents at the start.
- **Dynamic content last**: The latest user message and fresh tool results at the end.

This ordering matters for **prompt caching**: providers can reuse the processed prefix of a prompt across requests, cutting cost and latency, but only if the prefix is identical. Putting a timestamp or user-specific data at the top breaks the cache on every call.

For quality, put the most important instructions where the model attends best. Repeating a key constraint near the end of a long prompt often helps.

---

## Debugging Context

When output is wrong, look at the exact input before changing anything.

- **Log the full prompt** the model actually received, not the template.
- **Check what was retrieved** and whether the needed fact was present.
- **Measure token usage by section** (system, tools, history, retrieval) to see what's crowding the window.
- **Test removal**: Delete sections and see if quality changes. Often, a lot of context does nothing.
- **Watch for contradictions** between the system prompt, retrieved documents, and earlier conversation.

---

## Interview Questions

1. **What is context engineering, and how is it different from prompt engineering?**
   Prompt engineering is phrasing the instruction. Context engineering is managing everything the model sees on each call: instructions, examples, retrieved data, tools, tool results, history, and memory. It matters most for agents, where accumulated context dominates.

2. **Why can more context make answers worse?**
   Models don't use long context uniformly. Information in the middle gets missed, irrelevant content distracts, and reliability drops as length grows. More context also adds cost and latency.

3. **An agent does well early in a session and degrades later. Why, and how do you fix it?**
   Its context fills with stale tool results and history, diluting what matters. Fix it with compaction, clearing old tool results, external notes, and sub-agents for exploratory subtasks.

4. **What belongs in the system prompt vs. retrieved at runtime?**
   Stable role, rules, constraints, and output format go in the system prompt. Facts that are large, changing, or only sometimes relevant are retrieved, either up front or just in time through tools.

5. **How do you handle tasks bigger than the context window?**
   Retrieve selectively, summarize and compact history, keep durable notes outside the window, and split work across sub-agents that return condensed results.

6. **How does prompt ordering affect caching and cost?**
   Caching reuses an identical prefix. Put stable content (system prompt, tools, static docs) first and dynamic content last; anything that changes near the top invalidates the cache.

---

## References

- [Anthropic, Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Liu et al., Lost in the Middle: How Language Models Use Long Contexts (2023)](https://arxiv.org/abs/2307.03172)
- [Chroma, Context Rot: How Increasing Input Tokens Impacts LLM Performance (2025)](https://research.trychroma.com/context-rot)
