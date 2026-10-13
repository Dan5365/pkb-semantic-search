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

All referenced sources are freely accessible (open access / arXiv):

1. pgvector: https://github.com/pgvector/pgvector
2. Karpukhin et al. (2020). Dense Passage Retrieval for Open-Domain Question Answering. https://arxiv.org/abs/2004.04906
3. Cormack, Clarke, Büttcher (2009). Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods. SIGIR 2009.
4. Reimers & Gurevych (2019). Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. https://arxiv.org/abs/1908.10084
5. Malkov & Yashunin (2018). Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs. https://arxiv.org/abs/1603.09320
6. Robertson & Zaragoza (2009). The Probabilistic Relevance Framework: BM25 and Beyond. https://www.ftweb.org/pdf/bm25.pdf
7. Pan et al. (2023). Survey on Vector Database Management Systems. https://arxiv.org/abs/2310.11703
8. Manning, Raghavan, Schütze. Introduction to Information Retrieval. https://nlp.stanford.edu/IR-book/information-retrieval-book.html

## Quick Start

Setup instructions will be added once the first service is merged into the main branch.
cd pkb-semantic-search
docker-compose up --build -d
```

The baseline Auth and Notes services start from this repository. The search engine and frontend will be available after they are merged into the main branch.
