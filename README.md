# PKB Semantic Search

Intelligent Personal Knowledge Base: semantic search module on PostgreSQL/pgvector with an investigation into search strategy effectiveness.

**English:** Development of a Semantic Search Module for a Personal Knowledge Base Based on PostgreSQL/pgvector with an Investigation into Search Strategy Effectiveness

**Қазақша:** PostgreSQL/pgvector негізіндегі дербес білім базасына арналған семантикалық іздеу модулін әзірлеу және іздеу стратегияларының тиімділігін зерттеу

**Русский:** Разработка модуля семантического поиска для персональной базы знаний на основе PostgreSQL/pgvector и исследование эффективности стратегий поиска

## Project Status

This repository was created for the diploma thesis. The project is in the design and early development stage. The following components are being developed:

- Semantic search engine (dense, sparse/BM25 and hybrid retrieval with Reciprocal Rank Fusion)
- Embedding service (model selection, chunking, caching, error handling)
- React frontend for search and strategy comparison
- Benchmarking of retrieval strategies (Precision@K, Recall@K, MRR)
- E2E and integration tests, CI/CD pipeline

## Problem Statement

Personal knowledge bases accumulate large volumes of unstructured notes. Keyword-based search does not account for context, synonyms, or meaning. The project compares three retrieval strategies implemented on top of a single PostgreSQL database with the pgvector extension:

- **Sparse retrieval:** full-text search (FTS / BM25) in PostgreSQL.
- **Dense retrieval:** vector embeddings and similarity search via pgvector (cosine / L2 distance, HNSW indexes).
- **Hybrid retrieval:** Reciprocal Rank Fusion (RRF) of sparse and dense results.

## Team and Responsibilities

**Адильбекулы Даниял** (SE-2407), Lead Backend & Database Architect

PostgreSQL + pgvector schema design, HNSW/IVFFlat indexing, implementation of the three retrieval strategies and RRF ranking, REST API contract for the search service, core search service.

**Тапишев Даниял** (SE-2407), AI Integration & Frontend Engineer

Embedding model selection and justification of vector dimensionality, embedding generation service (chunking, caching, error handling), React UI for search results and strategy comparison, unit and UI tests for the frontend.

**Марат Ерканат** (SE-2407), QA, DevOps & Data Engineer

Unit, integration and E2E tests, CI/CD pipelines (GitHub Actions), Docker infrastructure, loading embeddings and data preparation, benchmarking of retrieval strategies (Precision@K, Recall@K, MRR).

**Academic supervisor:** Айтмұханбетова Эльвира Айтмұханбетқызы

**Project management:** YouTrack, [PKB project](https://pkb-team.youtrack.cloud/projects/PKB)

## Repository Structure

- `docs/` — project proposal, literature review, comparisons, requirements, diagrams, verification plan, interview results
- `services/` — backend services (Auth, Notes; Search Engine and Embedding Service are added during the thesis)
- `frontend/` — React application (added during the thesis)

## Tech Stack

- **Database and vector search:** PostgreSQL 16+, pgvector
- **Backend:** Python 3.11+, FastAPI, SQLAlchemy 2.0, Pydantic v2
- **Embeddings:** sentence-transformers (all-MiniLM-L6-v2)
- **Frontend:** React 19, Tailwind CSS, Vite
- **DevOps and testing:** Docker, Docker Compose, pytest, GitHub Actions

## Services

| Service | Port | Status |
|---|---|---|
| Search Engine | 8002 | planned |
| Embedding Service | 8003 | planned |
| Frontend | 5173 | planned |

## Literature

Sources are cited via arXiv or the publisher's DOI. Some sources may require institutional access.

1. Karpukhin, V., et al. (2020). Dense Passage Retrieval for Open-Domain Question Answering. EMNLP 2020. https://arxiv.org/abs/2004.04906
2. Cormack, G. V., Clarke, C. L. A., & Büttcher, S. (2009). Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods. In Proceedings of SIGIR 2009.
3. Reimers, N., & Gurevych, I. (2019). Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. https://arxiv.org/abs/1908.10084
4. Malkov, Y. A., & Yashunin, D. A. (2018). Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs. https://arxiv.org/abs/1603.09320
5. Robertson, S., & Zaragoza, H. (2009). The Probabilistic Relevance Framework: BM25 and Beyond. Foundations and Trends in Information Retrieval, 3(4), 333–389. https://doi.org/10.1561/1500000019
6. Pan, W., et al. (2023). Survey on Vector Database Management Systems. https://arxiv.org/abs/2310.11703
7. Manning, C. D., Raghavan, P., & Schütze, H. (2008). Introduction to Information Retrieval. Cambridge University Press. https://nlp.stanford.edu/IR-book/information-retrieval-book.html
8. pgvector: open-source PostgreSQL extension. https://github.com/pgvector/pgvector
9. Bruch, S., Gai, S., & Ingber, A. (2023). An Analysis of Fusion Functions for Hybrid Retrieval. ACM Transactions on Information Systems, 42(1). https://arxiv.org/abs/2210.11934
10. Thakur, N., Reimers, N., Rücklé, A., Srivastava, A., & Gurevych, I. (2021). BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models. NeurIPS 2021 Datasets and Benchmarks Track. https://arxiv.org/abs/2104.08663
11. Qu, R., Tu, R., & Bao, F. (2024). Is Semantic Chunking Worth the Computational Cost? https://arxiv.org/abs/2410.13070
12. Wang, L., Yang, N., Huang, X., Yang, L., Majumder, R., & Wei, F. (2024). Multilingual E5 Text Embeddings: A Technical Report. https://arxiv.org/abs/2402.05672

## Quick Start

Setup instructions will be added once the first service is merged into the main branch.

The baseline Auth and Notes services start from this repository. The search engine and frontend will be available after they are merged into the main branch.
