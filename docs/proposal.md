# Project Proposal

## Project Title

Development of a Semantic Search Module for a Personal Knowledge Base Based on PostgreSQL/pgvector with an Investigation into Search Strategy Effectiveness

## Team and Supervisor

- Adilbekuly Daniyal (SE-2407): backend and database architecture
- Tapishev Daniyal Beldekhanuly (SE-2407): AI integration and frontend
- Marat Erkanat Oskaruly (SE-2407): testing, DevOps and data preparation
- Supervisor: Aitmukhanbetova Elvira Aitmukhanbetkyzy, Senior Lecturer, School of Software Engineering

## Problem Statement

Personal knowledge bases accumulate large volumes of unstructured notes. Keyword-based search finds only exact matches and does not account for context, synonyms, or the meaning of a query. A user who remembers an idea but not its exact wording often cannot find the relevant note.

## Target Users

People who keep personal notes for months or years: students, researchers, developers, and other knowledge workers who use notes for study, work, and planning. These users search by meaning rather than by exact words.

Assumptions about user needs will be checked through interviews with 3–5 personal note-takers (see `docs/interviews.md`). The interview results separate confirmed findings from assumptions.

## Aim

Build a semantic search module for a personal knowledge base on a single PostgreSQL database with the pgvector extension, and experimentally compare three search strategies: full-text search (BM25), vector search (dense retrieval), and hybrid search (Reciprocal Rank Fusion).

## Objectives

1. Review the literature on dense retrieval, sparse retrieval, hybrid search, and vector indexes, including a comparison of existing note-search systems.
2. Select an embedding model and justify the vector dimensionality and the chunking strategy for notes.
3. Design the PostgreSQL schema with pgvector and configure vector indexes (HNSW, IVFFlat).
4. Implement the three search strategies and hybrid ranking with Reciprocal Rank Fusion.
5. Develop an interface that shows the results of all three strategies for the same query.
6. Evaluate search quality with Precision@K, Recall@K, and MRR on a labeled query set, and compare the strategies with each other and with a baseline solution.

## Expected Outcomes

- A working semantic search module with three strategies and an API.
- An interface for comparing the strategies on the same query.
- A labeled test set of queries with relevant notes.
- Evaluation results and conclusions about the conditions under which each strategy performs best.
- Documented architecture, requirements, verification plan, and development history in the repository.

## Project Scope

**In scope:**
- Search over the text notes of a single user.
- Three search strategies on PostgreSQL with pgvector and a comparison of their quality.
- A web interface for search and strategy comparison.
- Evaluation with metrics on a labeled test set of queries.

**Out of scope:**
- Mobile applications and native clients.
- Multi-user collaboration and synchronization across devices.
- Attachments (images, PDFs) and their indexing.
- Training custom embedding models: pretrained models are used.
- Production deployment and scaling to a large number of users.

## Constraints and Risks
- The project timeline is limited by the School's academic schedule; key results must be ready for the defense.
- The quality of the labeled test set affects the reliability of the metrics, since labeling is done manually.
- Results depend on the chosen embedding model; conclusions apply to the selected models and data.


