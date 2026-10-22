# PKB Semantic Search

Semantic search module for a personal knowledge base on PostgreSQL with the pgvector extension, and a comparison of three retrieval strategies on Russian-language personal notes.

**English:** Development of a Semantic Search Module for a Personal Knowledge Base Based on PostgreSQL/pgvector with an Investigation into Search Strategy Effectiveness

**Қазақша:** PostgreSQL/pgvector негізіндегі дербес білім базасына арналған семантикалық іздеу модулін әзірлеу және іздеу стратегияларының тиімділігін зерттеу

**Русский:** Разработка модуля семантического поиска для персональной базы знаний на основе PostgreSQL/pgvector и исследование эффективности стратегий поиска

## Project Status

At the pre-defense-1 stage the repository contains analysis, design and planning artifacts; implementation is in progress.

Planned components:

- Search module with three retrieval strategies: sparse (full-text search), dense (pgvector) and hybrid (Reciprocal Rank Fusion)
- Embedding service: model selection and comparison, chunking
- React interface for comparing strategies on one query
- Evaluation of retrieval strategies with Precision@K, Recall@K and MRR on a labeled query set
- Unit tests for the retrieval and metrics code

## Problem Statement

Users of a personal knowledge base often remember the meaning of a note but not its exact wording. Keyword search only matches exact terms and does not account for paraphrasing or synonyms. Personal notes are short and informal, while most retrieval methods have been evaluated on news and web text, so their effectiveness on this kind of data needs separate measurement.

The project compares three retrieval strategies implemented on one PostgreSQL database with the pgvector extension, on a corpus of Russian-language personal notes:

- **Sparse retrieval:** full-text search in PostgreSQL.
- **Dense retrieval:** embedding similarity search via pgvector (cosine distance, HNSW index).
- **Hybrid retrieval:** Reciprocal Rank Fusion (RRF) of the sparse and dense result lists.

The research question is which strategy returns relevant notes higher in the list, and under which conditions. The answer is based on a labeled query set and reported with Precision@K, Recall@K and MRR.

## Target Users

- Researchers and students who maintain personal note collections.
- Knowledge workers accumulating unstructured notes.
- Users who need search by meaning rather than by exact keywords.

## Expected Outcomes

- Working semantic search module on PostgreSQL/pgvector (sparse, dense, hybrid).
- Labeled query set and reproducible evaluation protocol.
- Comparative evaluation of the three strategies (Precision@K, Recall@K, MRR) on Russian-language personal notes.
- Comparison of pretrained embedding models for the personal-notes domain.
- Unit tests for the retrieval and metrics code.

## Team and Responsibilities

**Адильбекулы Даниял** (SE-2407), Backend and Database. Schema and migrations in PostgreSQL with pgvector, HNSW/IVFFlat indexes, the three retrieval strategies (sparse, dense, hybrid with RRF), the search API contract and the `/search` endpoint.

**Тапишев Даниял** (SE-2407), ML and Frontend. Embedding model selection and comparison, chunking, the embedding service with caching and error handling, the React interface for comparing strategies.

**Марат Ерканат** (SE-2407), Data Engineering and Evaluation. Labeled query set and labeling protocol, data loading and checks, baseline queries, Precision@K, Recall@K and MRR, the `/search/compare` endpoint, the view of experiment results.

**Academic supervisor:** Айтмұханбетова Эльвира Айтмұханбетқызы

**Project management:** YouTrack, [PKB project](https://pkb-team.youtrack.cloud/projects/PKB)

## Repository Structure

- `docs/`: project documents (proposal, requirements, literature review, comparisons, API examples, survey summary)
- `docs/diagrams/`: architecture diagrams as PlantUML sources (`.puml`) and exported images (`.png`)
- `docs/prototype/`: interface prototype screens
- `services/`: backend services (added during the thesis)
- `frontend/`: React application (added during the thesis)

## Documents

- [Project proposal](docs/proposal.md)
- [Requirements](docs/requirements.md)
- [Literature review](docs/literature.md)
- [Comparison of existing systems](docs/comparison_systems.md)
- [Comparison of technologies](docs/comparison_technologies.md)
- [API examples](docs/api_examples.md)

## Tech Stack

- **Database and vector search:** PostgreSQL 16+, pgvector
- **Backend:** Python 3.11+, FastAPI, SQLAlchemy 2.0, Pydantic v2
- **Embeddings:** sentence-transformers (pretrained models; all-MiniLM-L6-v2 as the baseline, alternatives are compared in the study)
- **Frontend:** React 19, Vite
- **Testing and tooling:** pytest, Docker, Docker Compose

## Services

| Service | Port | Status |
|---|---|---|
| Search Engine | 8002 | planned |
| Embedding Service | 8003 | planned |
| Frontend | 5173 | planned |

## Quick Start

Setup instructions will be added once the first service is merged into the main branch.
