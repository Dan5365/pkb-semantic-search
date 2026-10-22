# Embedding Model Selection

**Project:** PKB Semantic Search
**Author:** Tapishev Daniyal, SE-2407
**Status:** Draft. Candidate list and selection procedure. Measured results are added after the experiments.

## 1. Purpose

This document selects the embedding model and the chunking strategy for the dense retrieval strategy. It also justifies the vector dimensionality, because it affects storage, index size and search speed in pgvector. The choice is made on the labelled evaluation set (PKB-15), not on general leaderboards.

## 2. Requirements for the model

- **Language:** must handle Russian text. Models trained mainly on English are a valid candidate only as a control.
- **Runs locally:** no external API for the main experiments, to keep results reproducible and notes private (NFR-4, NFR-9).
- **Runs on a development machine without a GPU:** the reference environment is CPU (NFR-1).
- **Input length:** notes are split into chunks that fit the model's maximum input length.
- **Licence:** allows use in an academic project and publication of results.

## 3. Candidates

| Model | Type | Dimension | Languages | Max input (tokens) | Notes | Status |
|---|---|---|---|---|---|---|
| all-MiniLM-L6-v2 | local, sentence-transformers | 384 | mainly English | verify (about 256) | Small and fast; English-centred | verify |
| paraphrase-multilingual-MiniLM-L12-v2 | local, sentence-transformers | 384 | multilingual, includes Russian | verify | Same dimension as MiniLM, multilingual | verify |
| multilingual-e5-small | local | verify (384) | multilingual | verify | Requires text prefixes (`query:` / `passage:`); verify the prefix convention | verify |
| multilingual-e5-base | local | verify (768) | multilingual | verify | Larger; tests whether size helps | verify |
| Cloud embedding API (one provider, to be chosen) | remote API | verify per provider | multilingual | verify | Sends note text to a third party; per-request cost | verify |

The cloud model is included as a reference, not as the default. The main comparison is local.

## 4. Selection procedure

1. Split the labelled set (PKB-15) into a development part and a test part. Use the development part for chunk size and dimension choices. Use the test part only once, for the final numbers.
2. For each candidate, embed the same chunks and the same queries with the same chunking settings.
3. Compare on Recall@K and MRR for the dense strategy, and on the hybrid strategy as the main outcome (H1 and H2 in `docs/literature.md`).
4. Record time to embed the corpus and query latency on the reference CPU.
5. Choose the model that is best on the development part and report its result on the test part.

If the best multilingual model is only marginally better than the English model, report both and say so. The decision rule must be fixed before the experiment: a difference smaller than the agreed threshold is treated as no difference.

## 5. Chunking strategy

Candidates to compare on the development part:

- **Fixed-size chunks** with overlap (for example, 256 tokens with a small overlap). Simple and predictable.
- **Paragraph-based chunks.** Respect the note's own structure; short notes stay whole.
- **Whole note as one chunk** for notes shorter than the model limit. Baseline for the others.

The chosen setting is recorded with the model, because the two interact: a long chunk in a model with a short input limit gets truncated silently.

## 6. Dimensionality

Dimension is fixed by the model (for example, 384 or 768). The choice between candidates therefore also chooses the dimension. Consequences to record:

- **Storage:** vectors of 384 dimensions take about half the space of 768, for the same number of chunks.
- **Index:** HNSW build time and memory grow with dimension. Check the pgvector limit for indexed vector columns and confirm the chosen dimension is within it (verify in pgvector documentation).
- **Speed:** query time grows with dimension and with the number of chunks.

The dimension is justified by these three numbers on the reference machine, not by the model's popularity.

## 7. Cloud versus local

| Criterion | Local model | Cloud API |
|---|---|---|
| Quality | measured on the same set | measured on the same set |
| Cost | zero per request; compute on the machine | per token, verify provider pricing |
| Hardware | CPU is enough for the small models (to be measured) | none on our side |
| Privacy | notes stay on the machine | notes are sent to a third party (conflicts with NFR-9 for personal notes) |
| Reproducibility | fixed model version | depends on the provider changing the model |
| Network dependence | none | required |

The local model is the default. A cloud model is compared only if the local candidates are clearly insufficient, and only on the test data without personal notes, or with explicit consent of the note authors.

## 8. Results

To be filled after the experiments. One row per candidate and chunking setting, with Recall@K, MRR, embedding time and query latency, on the development part. The final model's test result is reported once.

| Model | Chunking | Dimension | Recall@K (dev) | MRR (dev) | Embed time | Query latency |
|---|---|---|---|---|---|---|
| to be filled | | | | | | |

## 9. Decision

To be written after the results: chosen model, chunking, dimension, and the reason in one paragraph. The reason must refer to the measured numbers and the decision rule from section 4.

## 10. References

- Reimers & Gurevych (2019), Sentence-BERT. https://arxiv.org/abs/1908.10084
- Wang et al. (2024), Multilingual E5 Text Embeddings. https://arxiv.org/abs/2402.05672
- Qu, Tu & Bao (2024), Is semantic chunking worth the computational cost? https://arxiv.org/abs/2410.13070
- pgvector documentation: https://github.com/pgvector/pgvector
