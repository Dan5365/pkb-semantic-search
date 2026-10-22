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

**Адильбекулы Даниял** (SE-2407), Backend and Database. Schema and migrations in PostgreSQL with pgvector, HNSW/IVFFlat indexes, three retrieval strategies (sparse, dense, hybrid with RRF), search API contract and /search endpoint.

**Тапишев Даниял (SE-2407)**, ML and Frontend. Embedding model selection and comparison, chunking, embedding service with caching and error handling, React interface for comparing strategies.

**Марат Ерканат (SE-2407)**, Data Engineering and Evaluation. Labeled query set and labeling protocol, data loading and checks, baseline queries, Precision@K, Recall@K, MRR, /search/compare endpoint, experiment results view.

**Academic supervisor:** Айтмұханбетова Эльвира Айтмұханбетқызы

**Project management:** YouTrack, [PKB project](https://pkb-team.youtrack.cloud/projects/PKB)

## Repository Structure

- `docs/` — project proposal, literature review, comparisons, requirements, diagrams, verification plan, interview results
- `services/` — backend services (Search Engine and Embedding Service are added during the thesis)
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


## Quick Start

Setup instructions will be added once the first service is merged into the main branch.
