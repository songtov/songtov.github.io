---
author: Chiho Song
pubDatetime: 2026-06-11T06:10:00Z
title: After Prompt Engineering Comes Loop Engineering
slug: loop-engineering
featured: false
draft: false
tags:
  - ai-agent
  - workflow
  - llm
  - loop-engineering
  - ai-tools
description: A thought that designing loops where inputs, outputs, and feedback keep improving matters more than the skill of using an LLM well once.
lang: en
---

Prompt engineering was the first language for using LLMs well.

What to ask for, what context to give, what format to get the answer in. These questions still matter. But when you use LLMs every day, at some point you see a problem more important than writing a single good prompt.

**Is this task making the next task easier?**

I'd like to call this loop engineering.

---

## A Prompt Is a Request; a Loop Is a System

A prompt is a single request.

Summarize this paper. Review this code. Clean up these meeting notes. Find the cause of this error.

A loop, on the other hand, is a structure where requests and results change the environment.

When you summarize a paper, it goes into the wiki. What's in the wiki becomes context for the next question. When a question reveals a gap, the wiki gets reinforced. Important definitions become Anki cards. Cards you get wrong in Anki call for another explanation, and that explanation goes back into the notes.

One answer isn't the end; it becomes the condition for the next answer.

That's a loop.

---

## What Makes a Good Loop

I think a good LLM loop has a few conditions.

1. **State has to persist**

   If a good conversation evaporates, you have to start over from scratch. Outputs should remain in a form the next session can read again: files, notes, issues, tests, cards, logs.

2. **There has to be a place for feedback**

   Human edits, failed tests, repeated questions, and confusing flashcards should all become inputs to the loop. If feedback lives outside the system, the loop never closes.

3. **The next action should automatically get smaller**

   A good loop makes the next action small. Instead of starting from "What should I do?" every time, you should already have organized context and candidate actions.

4. **Quality criteria have to be repeatable**

   Writing needs a checklist, coding needs tests, studying needs recall, research needs citations. LLM output should come back in through repeatable criteria, not plausibility.

---

## Example: A Study Loop

In studying, a loop can look like this.

1. Put material into `raw/`.
2. The LLM organizes the key concepts into markdown notes.
3. Pull definitions, comparisons, timelines, and formulas from the notes as Anki card candidates.
4. The human reviews Anki and gets cards wrong.
5. Have the LLM explain the missed cards again.
6. The explanation is reflected in the notes.
7. Better cards are generated again.

What matters in this loop isn't the LLM giving one smart answer.

What matters is that **confusion becomes an input that improves the system.**

---

## Example: A Development Loop

Development is similar.

1. Write requirements as an issue or document.
2. The LLM creates an implementation plan.
3. Change the code.
4. Tests and lint fail.
5. The failure logs become input to the LLM again.
6. After the fix, the reason for the change stays in a document or PR description.
7. The next person reads that record and gets up to speed faster.

Here, a failing test isn't an obstacle; it's the loop's sensor. Code review is the same. Human feedback isn't scolding the LLM; it's a signal that makes the system more accurate.

---

## Agents Get Stronger on Top of Loops

A common mistake when adopting AI agents is to look only at "how far can the agent go on its own."

In practice, you have to look at it a bit differently.

**When this agent fails, does that failure make the next run better?**

Are failure logs kept? Are human corrections stored in a learnable form? Are repeated tasks promoted to skills? Is context organized? Can the next run go without repeating the same explanation?

An agent's intelligence isn't only inside the model. It's also inside the loop.

---

## The Sense of Loop Engineering

Loop engineering isn't a grand framework. It's closer to a very practical sense.

- Where will outputs be kept?
- What should the next run read first?
- In what format will human feedback come in?
- How will failures be recorded?
- When should repeated requests be extracted into a skill or script?
- By what criteria will output quality be checked?

As you answer these questions, the LLM stops being just a chat window and becomes part of your work environment.

---

## Conclusion

Prompt engineering is the skill of talking to an LLM.

Loop engineering is the skill of designing an environment that has an LLM in it.

What will matter more going forward is not "which prompt is best" but "which loop gets better over time." Strong models will keep coming. But there will be a growing gap between people who use strong models from scratch every time and people who make model output keep improving their own environment.

I find the latter more interesting.
