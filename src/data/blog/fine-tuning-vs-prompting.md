---
author: Chiho Song
pubDatetime: 2026-11-04T13:00:00Z
title: Fine-Tuning vs. Prompting for Applied AI Engineers
slug: fine-tuning-vs-prompting
featured: false
draft: false
tags:
  - applied-ai-engineer
  - fine-tuning
  - llm
  - interview
description: Why the answer to "should we fine-tune?" is usually "not yet," what fine-tuning actually changes, and the cases where it clearly wins.
lang: en
---

"Should we fine-tune?" is one of the most common questions product teams ask an Applied AI Engineer. It's also a favorite interview question. The usual right answer is "not yet," and the interesting part is explaining why.

![A four-step ladder: prompt engineering, few-shot examples, RAG, fine-tuning](/images/fine-tuning-vs-prompting.svg)

---

## The Decision Ladder

Each step costs more than the one below it, so climb only when the lower step stops working:

1. **Prompt engineering**: Clear instructions and structure. Minutes to change.
2. **Few-shot examples**: Show the model the format and style you want.
3. **RAG**: Add knowledge the model doesn't have (see [RAG fundamentals](/posts/rag-fundamentals/)).
4. **Fine-tuning**: Change the model's behavior by training on your examples.

Many teams jump to step 4 because it sounds like "real" ML. Most problems are solved at steps 1–3.

---

## What Fine-Tuning Changes

Fine-tuning is good at teaching **how** to respond: format, style, tone, and narrow task skills like classification or extraction.

It's bad at teaching **what is true right now**. If your goal is "answer questions about our docs," fine-tuning on the docs is the wrong tool. Facts learned in training can't be cited, go stale when docs change, and get mixed up with what the model already knew. That's a RAG problem.

A short way to say it in an interview: **RAG changes what the model knows; fine-tuning changes how it behaves.**

---

## The Methods, Briefly

- **Supervised fine-tuning (SFT)**: Train on input/output pairs. The most common form.
- **Preference tuning**: Train on pairs of better and worse answers. DPO (Rafailov et al., 2023) does this without a separate reward model, which made it much simpler than classic RLHF.
- **LoRA**: Instead of updating all weights, train small low-rank matrices added to existing layers (Hu et al., 2021). Far fewer trainable parameters, cheaper training, and small adapter files you can swap per task.
- **QLoRA**: LoRA on a 4-bit quantized base model (Dettmers et al., 2023), so large models can be fine-tuned on a single GPU.

---

## When Fine-Tuning Wins

- **Consistent format at high volume**: Thousands of calls a day that must follow a strict structure, where long prompts are expensive.
- **Smaller, cheaper, faster models**: Train a small model to do one task as well as a large one. This is often the strongest business case.
- **Narrow, well-defined tasks**: Classification, extraction, routing, with plenty of labeled examples.
- **Style that's hard to describe**: A brand voice that prompts can't pin down but examples can.

**Distillation** combines the first two: use a large model to generate high-quality outputs, then fine-tune a small model on them. You keep most of the quality at a fraction of the cost and latency.

---

## What It Really Costs

The training run is the cheap part. The expensive parts are:

- **Data**: Collecting, cleaning, and labeling examples. A few hundred high-quality examples often beat thousands of noisy ones.
- **Evals**: You need a held-out test set and a baseline to prove the fine-tune is better (see [LLM evals](/posts/llm-evals/)).
- **Regressions**: The model can get worse at things you didn't train on.
- **Maintenance**: When a better base model ships, your fine-tune may be obsolete, or need redoing.

If you can't build the eval, you're not ready to fine-tune.

---

## Interview Questions

1. **A team wants to fine-tune on their docs so the model can answer questions. What do you say?**
   Use RAG. Fine-tuning doesn't reliably add facts, can't cite sources, and goes stale. Fine-tune later if format or behavior is still the problem.

2. **When does fine-tuning beat prompting and RAG?**
   High-volume tasks needing consistent format, narrow tasks with good labeled data, hard-to-describe styles, and moving to a smaller model for cost or latency.

3. **What is LoRA?**
   Training small low-rank adapter matrices instead of all weights. Cheaper, faster, and adapters are small and swappable.

4. **How do you know a fine-tune was worth it?**
   Compare against the best prompt and RAG baseline on a held-out eval set, including quality, cost, and latency, and check for regressions on other tasks.

---

## References

- [Hu et al., LoRA: Low-Rank Adaptation of Large Language Models (2021)](https://arxiv.org/abs/2106.09685)
- [Dettmers et al., QLoRA: Efficient Finetuning of Quantized LLMs (2023)](https://arxiv.org/abs/2305.14314)
- [Rafailov et al., Direct Preference Optimization (2023)](https://arxiv.org/abs/2305.18290)
- [OpenAI, Fine-tuning guide](https://platform.openai.com/docs/guides/fine-tuning)
