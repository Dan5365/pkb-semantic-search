# Technology Comparison

**Project:** PKB Semantic Search
**Author:** Daniyal Adilbekuly, SE-2407
**Date:** 19 October 2026

## 1. Purpose

This document compares the three key technical decisions of the PKB Semantic Search module: where vectors are stored, which vector index is used, and how results of sparse and dense retrieval are combined. For each decision, the document lists the alternatives, the chosen option, and the reasons for the choice. It also states the trade-offs, which the verification plan (PKB-15) will measure.

The comparison is based on documented properties of each technology. It does not contain new benchmarks. Quality and latency will be measured on the project's own test set.

## 2. Decision 1: Vector Storage

| Option | Strengths | Limitations for this project |
|---|---|---|
| **pgvector in PostgreSQL** (chosen) | Vectors are stored in the same database as notes and metadata. Filtering by note attributes and joining with full-text search use one query engine. No second system to synchronise. | Less specialised than dedicated vector databases; large-scale tuning is limited. Not needed at the expected corpus size. |
| **Qdrant** | Dedicated vector database with its own filtering and payload model. Built for large vector collections. | Requires a separate service and synchronisation with PostgreSQL. Adds deployment and consistency concerns outside the scope of the thesis. |
| **FAISS** | Fast similarity search library, widely used in research. | A library, not a database. Persistence, metadata, and filtering must be implemented by the project. |

**Chosen:** pgvector. The project's central question is comparing sparse, dense, and hybrid retrieval inside one PostgreSQL database, so keeping all data in one place makes the comparison fair and reproducible. Dedicated vector databases would add a second system whose behaviour could affect the results.

## 3. Decision 2: Vector Index Type

| Option | Strengths | Limitations |
|---|---|---|
| **HNSW** (chosen) | Usually better recall at a given query latency. Works well without per-dataset tuning of partition parameters. | Slower to build and uses more memory than IVFFlat. |
| **IVFFlat** | Faster to build and smaller in memory. | Recall depends on the number of lists and probes, which must be tuned. Usually lower recall for the same query cost. Index must be rebuilt as data changes significantly. |

**Chosen:** HNSW. The corpus of personal notes is expected to be small to medium, so build time and memory are not the main constraints. Recall matters more for a retrieval comparison, because missing relevant notes would bias the metrics. HNSW also avoids a separate tuning step that IVFFlat requires.

**Trade-off to measure:** build time and memory use of HNSW against IVFFlat on the project corpus, and the effect of the index on Recall@K. Both values are recorded in the verification plan.

## 4. Decision 3: Combining Sparse and Dense Results

| Option | Strengths | Limitations |
|---|---|---|
| **Reciprocal Rank Fusion (RRF)** (chosen) | Uses only the positions of documents in each ranked list, so scores from different methods do not need calibration. Has a single constant k = 60, which Cormack et al. (2009) found effective across several collections. | Ignores the magnitude of the scores, so a very confident match in one list has the same weight as its rank suggests. |
| **Weighted sum of normalised scores** | Can use the magnitude of scores, and the weights can be tuned to a specific corpus. | Requires normalising scores from BM25 and cosine similarity, which have different distributions. Weights must be tuned, which needs labelled data and risks overfitting to the test set. |

**Chosen:** RRF. It needs no training data or weight tuning, which keeps the hybrid strategy comparable to the sparse and dense strategies. This matches the goal of the thesis: to compare methods without tuning the hybrid one more than the others. The weighted sum is kept as a possible extension if RRF results are unsatisfactory.

**Trade-off to measure:** whether RRF improves Precision@K and MRR over the sparse and dense strategies on the project test set. If it does not, the result is reported as is.

## 5. Summary

| Decision | Chosen option | Main reason | Measured in PKB-15 |
|---|---|---|---|
| Vector storage | pgvector in PostgreSQL | One system, fair comparison | Query latency at the corpus size |
| Vector index | HNSW | Recall matters for the comparison | Recall@K, build time, memory |
| Result fusion | Reciprocal Rank Fusion | No tuning, comparable to other strategies | Precision@K, Recall@K, MRR |

## 6. References

- Cormack, G. V., Clarke, C. L. A., & Büttcher, S. (2009). Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods. In *Proceedings of SIGIR 2009*.
- Malkov, Y. A., & Yashunin, D. A. (2018). Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs. https://arxiv.org/abs/1603.09320
- pgvector documentation: https://github.com/pgvector/pgvector
