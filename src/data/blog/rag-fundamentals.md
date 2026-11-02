---
author: Chiho Song
pubDatetime: 2026-11-02T16:00:00Z
title: RAG Fundamentals for Applied AI Engineers
slug: rag-fundamentals
featured: false
draft: false
tags:
  - applied-ai-engineer
  - rag
  - llm
  - interview
description: The full RAG pipeline from ingestion to evaluation, its failure modes, and the questions Applied AI Engineer interviews ask about it.
lang: en
---

Retrieval-Augmented Generation (RAG) is the most common production LLM pattern. It's also the most common system-design prompt in Applied AI Engineer interviews: "Design a chatbot over our internal docs."

The weak answer is "put the docs in a vector DB and pass the top results to the model." The strong answer walks through the whole pipeline, names where it fails, and explains how you'd measure it. This post is my notes for the strong answer.

---

## What RAG Is and When to Use It

RAG retrieves relevant external text at query time and puts it into the model's context before generating an answer. The idea comes from Lewis et al. (2020), who combined a retriever with a generator so the model could use knowledge that isn't in its weights.

Use it when:

- The knowledge changes often (docs, tickets, policies).
- The corpus is too large to fit in the context window, or too expensive to send every time.
- You need citations so users can verify answers.
- Different users can see different documents.

Compare it with the alternatives:

| Approach     | Good for                                        | Weak at                                      |
| ------------ | ----------------------------------------------- | -------------------------------------------- |
| RAG          | Fresh, large, permissioned knowledge; citations | Retrieval quality limits answer quality      |
| Long context | Small, stable corpora; reasoning across one doc | Cost and latency per call; no access control |
| Fine-tuning  | Style, format, task behavior; domain vocabulary | Injecting facts that change; citing sources  |

A good rule of thumb: **fine-tuning changes how the model behaves; RAG changes what the model knows right now.** They're not mutually exclusive.

---

## The Pipeline

1. **Ingestion**: Load documents and parse them into clean text. PDFs, tables, and HTML are where most of the mess lives.
2. **Chunking**: Split documents into retrievable units.
3. **Embedding**: Turn each chunk into a vector with an embedding model.
4. **Indexing**: Store vectors (and often a keyword index) with metadata such as source, date, and permissions.
5. **Retrieval**: Embed the query and fetch the top-k candidate chunks.
6. **Reranking**: Re-score candidates with a stronger, slower model and keep the best few.
7. **Generation**: Put the chunks into the prompt and instruct the model to answer from them and cite them.

Each stage has its own failure modes, which is why "the answer is wrong" is never enough information to debug a RAG system.

---

## Chunking

Chunk size is a trade-off between precision and context.

- **Too small**: A chunk loses the context needed to understand it. "Revenue grew 3%" means nothing without knowing which company and which quarter.
- **Too large**: The embedding averages over multiple topics, so it matches queries poorly, and each retrieved chunk wastes context tokens.

Common strategies:

- **Fixed size with overlap**: Simple baseline. Overlap reduces the chance a key sentence is cut in half.
- **Structure-aware**: Split on headings, sections, or code blocks so chunks follow the document's own boundaries.
- **Contextual chunks**: Prepend a short summary of where the chunk sits in the document. Anthropic's Contextual Retrieval uses an LLM to write this context for each chunk before embedding it.

There's no universal best size. Pick a baseline, then tune it against an evaluation set.

---

## Retrieval: Dense, Sparse, and Hybrid

- **Dense retrieval** uses embeddings. It matches meaning, so "reset my password" can find "account recovery steps."
- **Sparse retrieval** (BM25) matches terms. It's better for exact identifiers: error codes, product names, function names, ticket IDs.
- **Hybrid retrieval** runs both and merges the results, often with reciprocal rank fusion. In practice, it's a strong default because real queries mix concepts and identifiers.

**Reranking** adds a cross-encoder or LLM that scores each query-chunk pair directly. It's slower than vector search, so you retrieve broadly (say top 50–150) and rerank down to a handful.

These pieces add up. In Anthropic's Contextual Retrieval experiments, contextual embeddings reduced the top-20 retrieval failure rate by 35%. Adding contextual BM25 brought the reduction to 49%, and adding reranking brought it to 67%. That measures whether the right chunk was retrieved, not end-to-end answer quality, but it shows how much retrieval quality is on the table.

---

## Common Failure Modes

| Symptom                                | Likely cause                                           |
| -------------------------------------- | ------------------------------------------------------ |
| "I don't know" when the answer exists  | Retrieval miss: bad chunking, query/doc vocabulary gap |
| Confident but outdated answer          | Stale index; no re-ingestion on document updates       |
| Answer mixes up two similar documents  | Chunks lack context (which product, which version)     |
| Answer ignores the retrieved text      | Too many chunks dilute the context; weak instructions  |
| Citations that don't support the claim | No faithfulness check; model fills gaps from memory    |
| User sees content they shouldn't       | Permissions not enforced at retrieval time             |

The debugging habit that matters: **check retrieval first.** Look at what was retrieved for a failing query before touching the prompt. If the right chunk isn't there, no prompt will fix it.

---

## Evaluating RAG

Evaluate retrieval and generation separately.

**Retrieval metrics** (needs a labeled set of queries and relevant chunks):

- **Recall@k**: Is a relevant chunk in the top k?
- **MRR (Mean Reciprocal Rank)**: How high is the first relevant chunk ranked?
- **Context precision**: How much of what was retrieved is actually relevant?

**Generation metrics** (often scored with an LLM judge, as in Ragas):

- **Faithfulness**: Are the answer's claims supported by the retrieved context?
- **Answer relevance**: Does the answer address the question?

Before launch, build a small golden set from real user questions. After launch, log queries, retrieved chunks, and answers, and sample failures into the golden set. That turns evaluation into a loop instead of a one-time check.

---

## Production Concerns

- **Latency**: Embedding the query, searching, reranking, and generating add up. Cache embeddings and frequent queries; rerank only what you must.
- **Cost**: Fewer, better chunks cost less than many mediocre ones. Prompt caching helps when the same documents are reused.
- **Freshness**: Re-ingest on document change, not on a fixed schedule only. Store document versions and timestamps as metadata.
- **Access control**: Filter by the user's permissions **at retrieval time**, using metadata in the index. Filtering after generation is too late, because the model has already seen the text.
- **Observability**: Log the full trace (query, rewritten query, retrieved IDs, scores, final prompt, answer) so failures can be replayed.

---

## Interview Questions

1. **When would you choose RAG over fine-tuning or long context?**
   RAG for knowledge that is large, changing, permissioned, or needs citations. Fine-tuning for behavior and format. Long context for small, stable corpora where cross-document reasoning matters.

2. **How do you pick a chunk size?**
   Start from document structure and a reasonable default, then tune against recall@k on an evaluation set. Too small loses context; too large blurs embeddings and wastes tokens.

3. **Why use hybrid search?**
   Embeddings match meaning but miss exact tokens like error codes and IDs. BM25 catches those. Combining them covers both kinds of query.

4. **How do you evaluate a RAG system?**
   Separate retrieval (recall@k, MRR, context precision) from generation (faithfulness, answer relevance). Use a golden set before launch and grow it from production failures.

5. **The bot answers confidently but wrong. How do you debug it?**
   Pull the trace. Check whether the right chunk was retrieved. If not, it's a retrieval problem (chunking, query rewriting, hybrid search, reranking). If it was, it's a generation problem (too much context, weak instructions, no faithfulness check).

6. **How do you enforce per-user document permissions?**
   Store access metadata with each chunk and filter at retrieval time so unauthorized text never reaches the model.

---

## References

- [Lewis et al., Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks (2020)](https://arxiv.org/abs/2005.11401)
- [Anthropic, Introducing Contextual Retrieval](https://www.anthropic.com/engineering/contextual-retrieval)
- [Ragas documentation: metrics](https://docs.ragas.io/en/stable/concepts/metrics/)
