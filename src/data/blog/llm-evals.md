---
author: Chiho Song
pubDatetime: 2026-11-03T01:45:00Z
title: LLM Evals for Applied AI Engineers
slug: llm-evals
featured: false
draft: false
tags:
  - applied-ai-engineer
  - evals
  - llm
  - interview
description: How to define success, build an eval set, choose between code checks, LLM-as-judge, and human review, and put evals into the development loop.
lang: en
---

Almost every Applied AI system-design answer gets the same follow-up: "How would you know it works?"

Evals are the answer. They separate a demo from a product. Without them, every prompt change, model upgrade, or retrieval tweak is a guess. With them, it's an experiment with a result.

In the [RAG](/posts/rag-fundamentals/) and [embeddings](/posts/embeddings-vector-search/) posts, evaluation showed up as one section. This post is the whole topic.

---

## Why Evals Matter

LLM systems fail differently from traditional software:

- **Non-determinism**: The same input can produce different outputs. One good run proves little.
- **Silent regressions**: A prompt tweak that fixes one case can break five others without any error.
- **Moving parts you don't control**: Model versions change, providers update behavior, and your data drifts.
- **Fuzzy correctness**: Many outputs are "better or worse," not "right or wrong."

Evals turn these into numbers you can track. The goal isn't a perfect score. It's knowing, before you ship, whether a change made things better or worse.

---

## Start with Success Criteria

Before building anything, define what "good" means. Good criteria are specific, measurable, and tied to the real task. Anthropic's guidance frames it this way: measurable targets, multiple dimensions, and test cases that mirror the real task distribution, including edge cases.

Bad: "The bot should give good answers."

Better:

- Answers the question correctly on 90% of the golden set.
- Every factual claim is supported by a retrieved document.
- Refuses out-of-scope requests instead of guessing.
- Median latency under 3 seconds; cost under a set budget per conversation.

Most systems need several dimensions at once: correctness, faithfulness, safety, tone, latency, and cost.

---

## Types of Evals

| Type                  | How it works                                       | Good for                                       |
| --------------------- | -------------------------------------------------- | ---------------------------------------------- |
| Code-based assertions | Check format, JSON schema, keywords, regex, length | Structure, constraints, cheap regression tests |
| Reference-based       | Compare with an expected answer                    | Extraction, classification, short answers      |
| LLM-as-judge          | A model grades output against a rubric             | Open-ended quality, faithfulness, tone         |
| Human review          | Experts label outputs                              | Ground truth, calibrating judges, edge cases   |
| Online signals        | A/B tests, thumbs up/down, retries, escalations    | Real-world impact after launch                 |

A practical order: **use code checks wherever possible, LLM judges where you must, and humans to keep the judges honest.** Code checks are cheap, fast, and deterministic. Judges scale fuzzy evaluation. Humans define what "correct" means.

---

## Building an Eval Set

Where test cases come from:

1. **Real usage**: Logs, support tickets, search queries. The best source, because it reflects the real distribution.
2. **Domain experts**: Cases that matter but are rare in logs.
3. **Edge and adversarial cases**: Ambiguous questions, out-of-scope requests, prompt injection attempts, empty or very long inputs.
4. **Past failures**: Every production bug becomes a test case so it can't silently come back.

How big? Start small. A few dozen well-chosen cases catch a lot and are easy to review by hand. Grow toward hundreds as the product matures and you need to detect smaller differences. Coverage of important case types matters more than raw size.

Keep the eval set separate from anything you put in prompts as examples. Otherwise you're testing memorization.

---

## Metrics by Task

- **Classification and extraction**: Accuracy, precision, recall, F1, exact match.
- **RAG**: Retrieval recall@k and MRR; generation faithfulness and answer relevance.
- **Code generation**: Tests passed, pass@k.
- **Open-ended generation**: Rubric scores (for example, 1–5 per criterion) or pairwise win rate against a baseline.
- **Operational**: Latency (p50, p95), cost per request, error and refusal rates.

Prefer binary or small-scale judgments ("Is every claim supported? yes/no") over vague 1–10 scores. They're easier to grade consistently and easier to act on.

---

## LLM-as-Judge

A judge model reads the input, the output, and a rubric, then returns a score or a preference.

**Pointwise** grading scores one output at a time. **Pairwise** grading asks which of two outputs is better, which is often more reliable for comparing a new version with a baseline.

Zheng et al. (2023) found that strong judges like GPT-4 agreed with human preferences over 80% of the time, about as often as humans agreed with each other. They also documented biases that still matter:

- **Position bias**: Preferring the first (or second) answer. Fix: evaluate both orders.
- **Verbosity bias**: Preferring longer answers. Fix: rubrics that reward concision and correctness explicitly.
- **Self-enhancement bias**: Preferring outputs from the same model family. Fix: use a different judge model or calibrate against humans.

How to make a judge trustworthy:

1. Write a specific rubric with examples of each score.
2. Ask for reasoning before the verdict.
3. Label a sample by hand and measure judge-human agreement.
4. Re-check agreement when you change the judge model or rubric.

A judge you haven't validated against human labels is just another unverified model.

---

## Offline vs. Online

**Offline evals** run on a fixed set before shipping. They're for catching regressions and comparing options.

**Online evaluation** watches real traffic after shipping: A/B tests, user feedback, retries, escalations to humans, and sampled outputs graded by judges.

Both are needed. Offline evals miss the queries you didn't think of. Online signals arrive too late to prevent a bad release. The loop between them is the point: online failures become offline test cases.

### Evals in CI

Treat evals like tests:

- Run a fast subset (code checks plus a small judged set) on every PR that touches prompts, models, or retrieval.
- Run the full suite before releases and model upgrades.
- Track scores over time and fail the build on meaningful regressions.

This is eval-driven development: write the eval for a new behavior, watch it fail, change the system, watch it pass.

---

## Evaluating Agents

Agents add steps, tools, and state, so a single final-answer score isn't enough.

- **Outcome**: Did the task get done? (Was the ticket resolved, the file correct, the test passing?)
- **Trajectory**: Were the right tools called with the right arguments? Were there unnecessary or dangerous steps?
- **Efficiency**: Number of steps, tokens, time, and cost.
- **Safety**: Did it stay within its permissions? Did it ask for confirmation where required?

Run agent evals in a sandboxed environment with checkable end states, and run each task several times, since one success can be luck.

---

## Common Mistakes

- **Vibe checks only**: Trying a few prompts by hand and calling it done.
- **No baseline**: A score means nothing without the previous version's score to compare with.
- **Overfitting to the eval set**: Tuning prompts until the set passes, while real performance stays flat. Keep a held-out set.
- **Eval set rot**: The product changes but the test cases don't.
- **Measuring what's easy instead of what matters**: High format compliance, low usefulness.
- **Unvalidated judges**: Trusting an LLM judge without checking it against humans.

---

## Interview Questions

1. **How would you evaluate a customer-support chatbot before launch?**
   Define success criteria (resolution accuracy, faithfulness to docs, correct refusals, latency, cost). Build a golden set from real tickets plus edge cases. Use code checks for format and policy, a validated LLM judge for correctness and tone, and human review on a sample. Compare against a baseline.

2. **When is LLM-as-judge appropriate, and how do you know it's reliable?**
   When outputs are open-ended and code checks can't capture quality. Validate by labeling a sample by hand and measuring agreement; control for position and verbosity bias; re-validate when the judge or rubric changes.

3. **How big should an eval set be?**
   Start with a few dozen high-signal cases you can review by hand, then grow to hundreds as you need to detect smaller changes. Coverage of important case types matters more than size.

4. **A new model scores higher on your eval but users complain. What happened?**
   The eval set probably doesn't match real traffic, or it measures the wrong thing. Pull the complaints, turn them into test cases, check for distribution drift, and check whether the judge is biased toward the new model's style.

5. **How do you evaluate a multi-step agent?**
   Score the outcome in a sandbox with a checkable end state, inspect the trajectory (tool choice, arguments, unnecessary steps), track cost and step count, and run each task multiple times.

6. **How do you put evals into CI/CD?**
   Run a fast subset on every PR that touches prompts, models, or retrieval; run the full suite before releases; track scores over time; fail on meaningful regressions.

---

## References

- [Zheng et al., Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena (2023)](https://arxiv.org/abs/2306.05685)
- [Anthropic, Define success criteria and build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)
- [Hamel Husain, Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)
