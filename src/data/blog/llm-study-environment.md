---
author: Chiho Song
pubDatetime: 2026-05-03T00:00:00Z
title: Setting Up a Study Environment with LLMs
slug: llm-study-environment
featured: true
draft: false
tags:
  - claude
  - cowork
  - workflow
  - learning
  - anki
  - ai-tools
description: Setting up a study environment on top of Claude OS. An idea for tying note-taking, Anki memorization, and web search into one loop.
lang: en
---

In my previous post, I wrote about using Claude like an operating system. That was about the substrate. This post is a proposal for **what to put on top of it.**

When developers get a new machine, the first thing they do is set up their environment: dotfiles, shell, editor, keybindings. They get it into a state that's good for work. Studying is the same. Now that LLMs are here, the bottleneck in studying isn't access to knowledge, it's **where knowledge flows.** Designing that flow is setting up the environment.

---

## Three Pillars of Studying

With LLMs in the picture, I broke studying back down into its components.

- **Note-taking**: Compiling outside knowledge into my own words.
- **Memorization**: Defending against forgetting. Some things ultimately have to live in your head.
- **External calls**: When you don't know something, ask and fetch it immediately. LLMs, web search, MCP.

If these three run separately, studying dies. Notes get written once and never read, memorization happens without context, and searches evaporate. They have to be tied into one **loop.**

---

## Environment = One Project per Domain

Here's the direction I'm heading. With Claude Cowork as HQ, I create one project (folder) per study topic: AI, math, Korean history, and so on.

Each project has:

- **Context + Summary**: How much I know in this domain and what I want to learn. The LLM reads this first at the start of every session.
- **Skills**: Definitions of repeated tasks. For example: "Take in one paper → summary + backlinks + extract Anki card candidates."
- **MCP / plugins**: Connections to external systems such as Anki MCP, web search, and Obsidian.

A single folder becomes a **personal environment** for that domain.

---

## Anki: Where You Outsource Forgetting

The LLM remembers the notes for you. But some things have to get into your own head: definitions, key formulas, dates, names, events. Things you need to pull out instantly, whether in an exam or in the middle of a conversation.

Plug in Anki MCP here and the loop closes.

1. Notes accumulate in the wiki.
2. The LLM turns important items into Anki cards.
3. They get pushed to a deck via MCP.
4. The human reviews Anki every day.
5. Cards that cause confusion go back to the LLM as questions → the wiki is reinforced → the cards are updated.

**Note-taking and memorization both live inside one loop.** That's the key.

---

## Any Domain Works

The same environment can be instantiated per domain.

- **AI**: Drop a paper PDF into `raw/` and call a skill, and it gets folded into the wiki. A few days later, a question like "Explain the difference between RoPE and ALiBi again" comes back answered in the tone of my own notes.
- **Math**: Write down definitions, lemmas, and proofs, and the LLM builds a dependency graph between definitions. Anki cards have the definition name on the front and the content plus intuition on the back. When a proof gets stuck, step-by-step hints also accumulate in the wiki.
- **Korean history**: There's a mountain of things to memorize, and the context is scattered. Ask about events and people with an LLM plus web search, and collect the summaries in the wiki. Timelines and names go to Anki, and context gets recalled from the wiki.

Different domains, same environment shape: context + skills + MCP + Anki.

---

## The Environment Is Half of Studying

Before LLMs, studying was a game of "how much did you read and memorize." After LLMs, it becomes a game of "**how well did you build a closed loop for knowledge to flow through.**"

Notes meet external search, Anki cards fall out of that, and whatever trips you up in Anki flows back into the notes. All inside one folder.

The OS is installed. Now it's time to lay down your own study environment on top of it.
