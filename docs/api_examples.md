# API Examples: Search Engine

**Project:** PKB Semantic Search
**Author:** Daniyal Adilbekuly, SE-2407
**Status:** Proposed. The response fields must be agreed with the frontend (Tapishev) before implementation.

Base URL of the Search Engine service: `http://localhost:8002`

## 1. Endpoints

| Method | Path | Purpose | Requirement |
|---|---|---|---|
| POST | `/search` | Search notes with one strategy (sparse, dense, hybrid) | FR-8 |
| POST | `/search/compare` | Run all three strategies for the same query | FR-9 |
| GET | `/health` | Service and dependency status | NFR-6 |

## 2. Search with one strategy

**Request**

```json
{
  "query": "where did I write about index tuning",
  "strategy": "hybrid",
  "k": 5
}
```

Fields:
- `query` (string, required): the user query, 1–500 characters.
- `strategy` (string, required): one of `sparse`, `dense`, `hybrid`.
- `k` (integer, optional, default 10): number of results, from 1 to 50.

**Response, 200 OK**

```json
{
  "query": "where did I write about index tuning",
  "strategy": "hybrid",
  "k": 5,
  "status": "ok",
  "results": [
    {
      "rank": 1,
      "note_id": "note_042",
      "chunk_id": "chunk_0137",
      "score": 0.0328,
      "excerpt": "HNSW parameters m and ef_construction control the trade-off between recall and build time..."
    },
    {
      "rank": 2,
      "note_id": "note_017",
      "chunk_id": "chunk_0051",
      "score": 0.0318,
      "excerpt": "Tuning the index for the query workload means measuring latency at different ef_search values..."
    }
  ],
  "model": "all-MiniLM-L6-v2",
  "index": "hnsw"
}
```

`score` for `hybrid` is the Reciprocal Rank Fusion score. For `sparse` it is the BM25 score, for `dense` the cosine similarity. Scores are comparable only within one strategy.

**Response, 400 Bad Request** (invalid strategy)

```json
{
  "error": "invalid_strategy",
  "message": "strategy must be one of: sparse, dense, hybrid",
  "allowed": ["sparse", "dense", "hybrid"]
}
```

## 3. Degraded mode (NFR-3)

If the embedding service is unavailable, dense and hybrid requests return sparse results and report the degradation. The HTTP status stays 200.

```json
{
  "query": "where did I write about index tuning",
  "strategy": "hybrid",
  "k": 5,
  "status": "degraded",
  "reason": "embedding_service_unavailable",
  "fallback": "sparse",
  "results": [
    {
      "rank": 1,
      "note_id": "note_042",
      "chunk_id": "chunk_0137",
      "score": 4.21,
      "excerpt": "..."
    }
  ]
}
```

## 4. Compare three strategies (FR-9)

**Request**

```json
{
  "query": "where did I write about index tuning",
  "k": 5
}
```

**Response, 200 OK**

```json
{
  "query": "where did I write about index tuning",
  "k": 5,
  "status": "ok",
  "strategies": {
    "sparse": [
      { "rank": 1, "note_id": "note_017", "chunk_id": "chunk_0051", "score": 4.21, "excerpt": "..." }
    ],
    "dense": [
      { "rank": 1, "note_id": "note_042", "chunk_id": "chunk_0137", "score": 0.81, "excerpt": "..." }
    ],
    "hybrid": [
      { "rank": 1, "note_id": "note_042", "chunk_id": "chunk_0137", "score": 0.0328, "excerpt": "..." }
    ]
  }
}
```

The three lists use the same `k` and the same query, so the interface can show them side by side.

## 5. Health check

**Response, 200 OK**

```json
{
  "status": "ok",
  "database": "ok",
  "embedding_service": "ok",
  "index": "hnsw"
}
```

## 6. Open questions for the interface (to agree with Tapishev)

1. Is `excerpt` needed in full or truncated to a fixed length (proposed: 300 characters)?
2. Should the response include the note title? Proposed: not in MVP, add later if the interface needs it.
3. Pagination is out of scope for the MVP; `k` is the only limit.
