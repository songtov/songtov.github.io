---
author: Chiho Song
pubDatetime: 2026-11-02T19:45:00Z
title: Embeddings and Vector Search for Applied AI Engineers
slug: embeddings-vector-search
featured: false
draft: false
tags:
  - applied-ai-engineer
  - embeddings
  - vector-search
  - rag
  - interview
description: What embeddings measure, how approximate nearest neighbor indexes trade accuracy for speed, and how to choose and operate them in production.
lang: en
---

Embeddings sit under almost every Applied AI system: RAG, semantic search, deduplication, clustering, and recommendation. In the [RAG fundamentals post](/posts/rag-fundamentals/), retrieval was one box in the pipeline. This post opens that box.

Interviewers ask about embeddings to see whether you understand what similarity actually measures, how approximate search trades accuracy for speed, and how you'd choose a model instead of defaulting to whatever is popular.

---

## What an Embedding Is

An embedding model maps an input (text, an image, code) to a fixed-length vector. The model is trained so that inputs with similar meaning end up close together in that vector space.

Two things follow from this:

- **"Similar" means what the training data made it mean.** A general-purpose model may place "Python" near "snake" and "Java" near "coffee" more than you'd like for a code-search tool.
- **Vectors from different models are not comparable.** Distances are only meaningful inside one model's space.

---

## Similarity Metrics

| Metric            | What it measures       | Notes                              |
| ----------------- | ---------------------- | ---------------------------------- |
| Cosine similarity | Angle between vectors  | Ignores length; the common default |
| Dot product       | Angle and length       | Fastest; length can carry signal   |
| Euclidean (L2)    | Straight-line distance | Sensitive to vector length         |

The key fact: **if vectors are normalized to unit length, all three give the same ranking.** For unit vectors, the dot product equals cosine similarity, and squared Euclidean distance is `2 - 2 × cosine`. Many embedding models output normalized vectors, which is why the choice often doesn't matter in practice. Use the metric the model was trained with; the model card usually says.

---

## Choosing an Embedding Model

Things to compare:

- **Domain fit**: Legal, medical, code, and multilingual text often need a specialized or multilingual model.
- **Max input length**: Text past the limit is truncated, which silently changes what you index.
- **Dimensions**: More dimensions can mean better quality but more memory and slower search. Some models are trained so you can truncate vectors to fewer dimensions with modest quality loss (Matryoshka representation learning).
- **Cost and latency**: Hosted API vs. self-hosted, price per token, throughput for bulk re-embedding.
- **Asymmetric inputs**: Some models expect different prefixes or modes for queries and documents. Getting this wrong quietly hurts retrieval.

**MTEB** (Massive Text Embedding Benchmark) is a good place to shortlist models, but it's a starting point, not the answer. Leaderboard averages hide large differences by task and domain. The real test is recall@k on a labeled set built from your own queries and documents.

---

## Exact vs. Approximate Search

**Exact (brute-force) search** compares the query with every vector. It gives perfect recall and is fine up to tens or hundreds of thousands of vectors, but cost grows linearly with the corpus.

**Approximate nearest neighbor (ANN)** indexes give up a little recall for much faster search:

- **HNSW (Hierarchical Navigable Small World)**: A layered graph. Search starts at a sparse top layer, greedily moves toward the query, and descends to denser layers. Key parameters:
  - `M`: links per node. Higher means better recall and more memory.
  - `ef_construction`: candidate list size while building. Higher means a better graph and slower builds.
  - `ef_search`: candidate list size at query time. The main knob for trading latency against recall.
- **IVF (Inverted File)**: Cluster vectors with k-means, then search only the nearest clusters. Faster to build and lighter on memory than HNSW; recall depends on how many clusters you probe.
- **Product quantization (PQ)**: Compress vectors into short codes to cut memory, at some accuracy cost. Often combined with IVF.

The interview framing: **every ANN index is a trade-off among recall, latency, memory, and build time.** You tune it by measuring recall against brute-force results on a sample.

---

## Do You Need a Vector Database?

Often, no. A vector index inside a database you already run (for example, Postgres with pgvector, which supports HNSW and IVFFlat indexes) gives you:

- Transactions and joins with your existing data
- One system to back up, secure, and monitor
- Permissions and metadata in the same place as the vectors

A dedicated vector database earns its place at large scale, with very high query throughput, or when you need features like built-in hybrid search, multi-tenancy, or managed sharding. Start with what you have, and move when measurements say so.

---

## Metadata Filtering

Real queries come with filters: "only this customer's documents," "only docs updated this year." Filtering interacts badly with ANN.

- **Post-filtering**: Run ANN for top-k, then drop rows that fail the filter. If the filter is selective, you can end up with far fewer than k results, or none.
- **Pre-filtering**: Restrict to matching rows first, then search. Accurate, but if the filtered set is large and the index can't use the filter, you may fall back to slow scans.

Systems handle this differently. pgvector 0.8.0, for example, added iterative index scans (`hnsw.iterative_scan`), which keep scanning the index until enough rows pass the filter, up to a configurable limit. For access control, the filter must never be optional: wrong results are bad, leaked documents are worse.

---

## Evaluating Embeddings on Your Data

1. Collect real queries (logs, support tickets, or written by domain experts).
2. Label the relevant documents for each query.
3. Measure recall@k and MRR for each candidate model and index configuration.
4. Compare ANN results with brute-force results to measure the recall lost to approximation.

This separates two questions that often get mixed up: "Is the model good for my domain?" and "Is my index configured well?"

---

## Operational Issues

- **Changing models means re-embedding everything.** Old and new vectors can't be mixed. Plan for a parallel index, a backfill, and a cutover.
- **Version your embeddings.** Store the model name and version with each vector.
- **Watch chunking changes too.** New chunk boundaries also require re-embedding.
- **Budget memory.** Vectors times dimensions times 4 bytes (float32), plus index overhead. One million 1,536-dimension float32 vectors is about 6 GB before the index.

---

## Interview Questions

1. **Why cosine similarity instead of Euclidean distance?**
   Cosine ignores vector length and compares direction, which is usually what carries meaning. For unit-normalized vectors, cosine, dot product, and Euclidean distance give the same ranking, so use whatever the model was trained with.

2. **How does HNSW work, and what do its parameters trade off?**
   It builds a layered proximity graph and searches greedily from sparse to dense layers. `M` and `ef_construction` trade memory and build time for graph quality; `ef_search` trades query latency for recall.

3. **How would you choose an embedding model for a new domain?**
   Shortlist from MTEB and domain-specific options, check input length, dimensions, language, and cost, then pick based on recall@k on a labeled set from the real domain.

4. **What happens when you switch embedding models in production?**
   Every vector must be re-embedded because the spaces aren't compatible. Build the new index in parallel, backfill, evaluate, then cut over, with model versions stored per vector.

5. **Pre-filtering vs. post-filtering: what breaks?**
   Post-filtering can return too few results when the filter is selective. Pre-filtering is accurate but can be slow if the index can't use the filter. Use engine features like filtered or iterative index scans, and never treat access-control filters as optional.

6. **Do you need a dedicated vector database?**
   Not by default. An index in an existing database is simpler to operate and keeps metadata and permissions together. Move to a dedicated system when scale, throughput, or required features justify it.

---

## References

- [Malkov & Yashunin, Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs](https://arxiv.org/abs/1603.09320)
- [Muennighoff et al., MTEB: Massive Text Embedding Benchmark](https://arxiv.org/abs/2210.07316)
- [Kusupati et al., Matryoshka Representation Learning](https://arxiv.org/abs/2205.13147)
- [pgvector README](https://github.com/pgvector/pgvector)
