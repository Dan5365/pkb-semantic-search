# Requirements Specification
**Project:** PKB Semantic Search **Author:** Daniyal Adilbekuly

## 1. Scope and Context

The system is a semantic search module for a personal knowledge base. It indexes the text notes of a single user in PostgreSQL with the pgvector extension and supports three retrieval strategies: sparse (BM25 / full-text search), dense (vector similarity), and hybrid (Reciprocal Rank Fusion). The module is evaluated on a labeled query set. Out-of-scope items are listed in `docs/proposal.md`.

Priorities use the MoSCoW scheme: **M** = must have, **S** = should have, **C** = could have.

## 2. Actors

- **User:** a person who keeps personal notes and searches them.
- **Evaluator:** a team member who runs the experiments and reads the metrics.
- **Client application:** the web interface that calls the search API.

## 3. Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-1 | The system shall store notes as text documents, each with a unique identifier, title, body, and creation time. | M |
| FR-2 | The system shall split each note into chunks for indexing according to a documented chunking strategy. | M |
| FR-3 | The system shall generate an embedding vector for each chunk using the selected embedding model. | M |
| FR-4 | The system shall store embeddings in PostgreSQL with pgvector and build a vector index (HNSW or IVFFlat). | M |
| FR-5 | The system shall support sparse retrieval using full-text search (BM25 or equivalent ranking) over the same notes. | M |
| FR-6 | The system shall support dense retrieval by cosine or L2 similarity between the query embedding and chunk embeddings. | M |
| FR-7 | The system shall support hybrid retrieval by combining the sparse and dense result lists with Reciprocal Rank Fusion. | M |
| FR-8 | The API shall accept a query, a strategy identifier (sparse, dense, hybrid), and a result limit K, and return a ranked list of notes with scores. | M |
| FR-9 | The API shall return the results of all three strategies for the same query in a single response, to support comparison. | S |
| FR-10 | The web interface shall let the user enter a query, choose a strategy, and view the ranked results. | M |
| FR-11 | The web interface shall display the results of the three strategies side by side for the same query. | S |
| FR-12 | The system shall allow adding, updating, and deleting notes, and re-indexing the changed chunks. | S |
| FR-13 | The system shall store the labeled query set: queries and the identifiers of relevant notes. | M |
| FR-14 | The evaluation module shall compute Precision@K, Recall@K, and MRR for each strategy on the labeled query set. | M |
| FR-15 | The evaluation module shall produce a report that compares the strategies with each other and with a baseline. | S |
| FR-16 | The system shall provide a baseline search (keyword search in PostgreSQL) for comparison in the evaluation. | M |
| FR-17 | The system shall support multiple embedding models selected by configuration, without code changes. | C |

## 4. Non-Functional Requirements

| ID | Category | Requirement | Priority |
|---|---|---|---|
| NFR-1 | Performance | For a corpus of 10,000 chunks on the development machine, the p95 latency of a single query shall not exceed 500 ms for dense and hybrid retrieval. | M |
| NFR-2 | Performance | Indexing of 1,000 notes shall complete within 15 minutes on the development machine. | S |
| NFR-3 | Reliability | If the embedding model is unavailable, the system shall return sparse results and report that dense retrieval is degraded. | S |
| NFR-4 | Reproducibility | Every experiment shall be reproducible from the repository: the model version, chunking parameters, index parameters, and the query set version shall be recorded. | M |
| NFR-5 | Maintainability | Each service shall have automated unit tests, and the API shall be covered by integration tests. | M |
| NFR-6 | Deployability | The backend services and the database shall start with a single documented command using Docker Compose. | M |
| NFR-7 | Security | API keys and credentials shall not be stored in the repository; configuration shall be read from environment variables. | M |
| NFR-8 | Documentation | The API shall be documented with OpenAPI. The README shall describe how to run and test the system. | M |
| NFR-9 | Privacy | Notes used in experiments shall be either synthetic or collected with the consent of their authors, and shall not be published in the repository. | M |

## 5. Acceptance Criteria

Each requirement is accepted when the following conditions are met.

| Requirement | Acceptance criterion |
|---|---|
| FR-4 | An HNSW or IVFFlat index exists on the embedding column, and a query uses it, confirmed by `EXPLAIN` output. |
| FR-5, FR-6, FR-7 | For a test query, each strategy returns a ranked list of K notes, and the hybrid list is the RRF combination of the sparse and dense lists. |
| FR-8 | A request with a valid query and strategy returns HTTP 200 and a JSON body that matches the OpenAPI schema. An invalid strategy returns HTTP 400. |
| FR-9, FR-11 | The response or screen shows the three strategies for one query, with the same K for each. |
| FR-14 | The evaluation script outputs Precision@K, Recall@K, and MRR for each strategy on the labeled query set, and the values match an independent recomputation on a small sample. |
| FR-13 | The labeled query set contains the agreed number of queries, and each query has at least one relevant note. |
| NFR-1 | A load test on the agreed corpus shows p95 latency at or below 500 ms for dense and hybrid retrieval. |
| NFR-6 | On a clean machine with Docker, the documented command starts all services, and the health endpoints respond. |
| NFR-7 | A search of the repository for secrets finds no API keys or passwords in committed files. |

The numeric thresholds in NFR-1 and the corpus size are proposed values. The team should confirm them after the first measurements.

## 6. Constraints and Assumptions

- Embedding models are pretrained and used as they are.
- The corpus is text only.
- The development machine is the reference environment for performance measurements.
- Interview results (`docs/interviews.md`) are used to check the assumptions about user needs. Requirements that depend on unconfirmed assumptions are marked in the document.
