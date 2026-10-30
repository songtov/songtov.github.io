---
author: Chiho Song
pubDatetime: 2026-05-02T00:00:00Z
title: Using Claude Like an Operating System
slug: claude-as-os
featured: true
draft: false
tags:
  - claude
  - cowork
  - workflow
  - obsidian
  - ai-tools
description: Building a personal AI OS with Karpathy's LLM knowledge base method and Claude Cowork.
lang: en
---

Andrej Karpathy recently shared his workflow. In short:

> Collect raw data, and the LLM "compiles" it into a `.md` wiki. Humans don't touch the wiki directly. It's the LLM's domain.

I added one more layer on top of this structure: using Claude like an operating system.

---

## The Structure Karpathy Described

1. **Ingest**: Gather articles, papers, repos, and images into a `raw/` directory.
2. **Compile**: The LLM reads `raw/` and organizes it into a `.md` wiki: summaries, backlinks, and per-concept articles.
3. **IDE**: Open the wiki in Obsidian to explore it. It's the human viewer for what the LLM wrote.
4. **Q&A**: Once the wiki is big enough, ask an LLM agent complex questions. Because the LLM maintains the index automatically, it works without RAG.
5. **Output**: Render answers as markdown, slides, or images and view them in Obsidian.
6. **Linting**: Periodically health-check the whole wiki: inconsistent data, missing information, candidates for new articles.

The core of the loop: **queries strengthen the wiki.** Every time I ask something, the result accumulates back into the wiki.

---

## The LLM OS View

There's a reason this structure looks like an OS.

- **LLM** = kernel. It coordinates work, calls tools, and transforms data.
- **Folders** = file system. The wiki and raw data are persistent storage.
- **Obsidian** = display. The viewer humans use to read what's in the file system.

Just as a traditional OS runs programs, this structure **runs intentions.**

---

## Implementing It with Claude Cowork

[Claude Cowork](https://claude.com/product/cowork) is the agent runtime built into Claude Desktop. It gives you the same agentic capabilities as Claude Code without a terminal.

The key idea is **treating a folder like a VM.**

1. **Create a `Claude OS/` folder**: This folder becomes an isolated workspace. Everything accumulates here: `raw/`, the wiki, email drafts, reports, and other outputs.
2. **Point Cowork at the folder**: Connect `Claude OS/` as the root of a Cowork project. Claude reads and writes only inside this folder.
3. **Define rules and workflows**: Write rules in the project instructions: file naming conventions, which tasks to handle in which format, schedules for recurring tasks, and so on.
4. **Run the agent**: Just describe the result you want. Claude makes a plan, runs parallel workstreams, and creates files directly. Open the results in Obsidian.

At that point, the `Claude OS/` folder is no longer just a directory. It becomes **a personal computing environment close to a VM**, where rules are specified, agents live, and outputs accumulate.
