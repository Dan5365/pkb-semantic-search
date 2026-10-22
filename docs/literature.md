# Literature Review

**Project:** PKB Semantic Search
**Author:** Daniyal Adilbekuly, SE-2407 (sections on embeddings and chunking to be reviewed with Tapishev, sections on metrics with Marat)

## 1. Question of the review

What is known about three retrieval strategies (sparse, dense, hybrid) and on which data they were evaluated? Which conditions are not covered, in particular short, informal personal notes in Russian?

## 2. Method

Sources come from the reference lists of known works (snowballing) and from searches in Google Scholar and arXiv with the queries listed in section 6. The search has not been run yet; the results will be recorded in section 6 with dates and counts.

Status column: **read** means the content was checked in the full text or in the sections cited; **verify** means the claims in the row have not been checked against the source and must not be used in the thesis until they are.

## 3. Summary of sources

| # | Source | Type | What it shows | Data and evaluation | Limitation for personal notes | Role in the project | Status |
|---|---|---|---|---|---|---|---|
| 1 | Karpukhin et al. (2020), DPR | EMNLP, peer-reviewed | Dense retriever trained with a dual encoder beats a Lucene-BM25 system by 9–19 points in top-20 passage retrieval accuracy on open-domain QA | Open-domain QA benchmarks built on Wikipedia; top-k retrieval accuracy | Factoid questions over encyclopedic text; supervised training on question–passage pairs; no personal notes | Motivates dense retrieval (strategy 2) | read (abstract) |
| 2 | Cormack, Clarke, Büttcher (2009), RRF | SIGIR, peer-reviewed | Reciprocal Rank Fusion with k = 60 outperforms individual systems and Condorcet Fuse; needs no training | TREC runs and LETOR 3 (583,850 document–query pairs); MAP and sign test | TREC news and web collections; k set on a pilot and not tuned later; no fusion on personal notes | Defines the hybrid strategy (strategy 3) | read (full text) |
| 3 | Robertson & Zaragoza (2009), BM25 and beyond | Monograph, Foundations and Trends in IR | Derivation of BM25 from the probabilistic relevance framework, including term-frequency saturation and length normalisation; typical ranges for k1 (1.2–2) and b (0.5–0.8) | Mostly theoretical, with references to TREC experiments; the authors state that the survey is not primarily experimental | Assumes term independence; positional information gives little average gain; parameters depend on the collection | Defines the sparse strategy (strategy 1) and its parameters | read (sections 3.4–3.5) |
| 4 | Reimers & Gurevych (2019), Sentence-BERT | EMNLP, peer-reviewed | Siamese BERT networks produce sentence embeddings compared with cosine similarity, much faster than cross-encoders | STS and transfer benchmarks, English | English only; sentences rather than chunked notes | Basis of the embedding model family (all-MiniLM-L6-v2) | verify |
| 5 | Malkov & Yashunin (2018), HNSW | arXiv; later journal version (IEEE TPAMI) | Graph-based approximate nearest neighbour index with logarithmic search complexity | Public ANN benchmarks; recall versus query speed | Generic vectors, not text embeddings of personal notes | Justifies the HNSW choice (PKB-5) | verify |
| 6 | Bruch, Gai, Ingber (2023), fusion functions for hybrid retrieval | ACM TOIS, peer-reviewed | Compares RRF with a weighted combination of scores; reports that a tuned weighted combination can outperform RRF | Public retrieval benchmarks, sparse and dense runs | Weights must be tuned on labelled data; relevant to our untuned RRF | Counter-evidence to RRF; must be discussed | verify |
| 7 | Thakur et al. (2021), BEIR | NeurIPS Datasets and Benchmarks | Zero-shot benchmark over heterogeneous datasets; BM25 is a strong baseline; dense models trained on one domain transfer unevenly | Multiple datasets, nDCG@10 and other metrics | No personal notes; mostly English | Shows that results depend on the domain; supports BM25 as baseline | verify |
| 8 | Qu, Tu, Bao (2024), semantic chunking | arXiv preprint | Studies whether semantic chunking is worth its computational cost | To verify | Not checked against personal notes | Background for the chunking experiment (Tapishev) | verify |
| 9 | Wang et al. (2024), Multilingual E5 | arXiv technical report | Multilingual embedding model for many languages | To verify: benchmarks and language coverage | General text; no personal notes | Candidate second model for the Russian comparison | verify |
| 10 | Pan et al. (2023), Survey on vector database management systems | arXiv survey | Overview of vector database designs and index types | Survey, no own experiments | Not focused on PostgreSQL extensions | Context for choosing pgvector over dedicated vector databases | verify |
| 11 | Manning, Raghavan, Schütze (2008), Introduction to IR | Textbook | Standard definitions of precision, recall, ranking and evaluation | Textbook | Not a research result | Definitions of Precision@K, Recall@K and MRR | verify |
| 12 | pgvector | Open-source documentation | Index types (HNSW, IVFFlat) and distance operators in PostgreSQL | Documentation, not scientific | Not peer-reviewed | Technical basis of the dense strategy | verify |

**Count.** Peer-reviewed sources: DPR, Cormack et al., Sentence-BERT, HNSW (journal version), Bruch et al., BEIR, and Robertson & Zaragoza (monograph). That is 7 if the HNSW journal version is confirmed. The requirement is at least 8 scientific sources, so one more peer-reviewed work is needed, or the supervisor must confirm that arXiv preprints count.

## 4. Themes from the sources

**Dense retrieval outperforms keyword search on open-domain question answering (DPR).** The reported gain is measured on Wikipedia passages and questions with supervised training. It does not transfer automatically to other domains.

**Outside the training domain, keyword search remains a strong baseline (BEIR).** BM25 performs competitively across heterogeneous datasets, while dense models trained on one domain transfer unevenly. This is the main reason to keep BM25 as the sparse strategy and as the baseline.

**Fusion is a simple default, but tuning is an open question (Cormack et al., Bruch et al.).** RRF needs no training and combines methods by rank. Bruch et al. report that a tuned weighted combination can do better. Our study keeps RRF untuned, so the comparison is fair across strategies, but the thesis must state this limitation and discuss the tuned alternative.

**Chunking and model choice change what the embedding represents (Qu et al., Wang et al.).** These works are cited as background for the chunking and model experiments. Their findings must be read and summarised before this theme is used in the thesis.

**The gap.** The reviewed sources evaluate retrieval mainly on encyclopedic, news and web text, mostly in English, with question-style queries. None of the reviewed sources, as far as the reading so far shows, compares sparse, dense and hybrid retrieval on personal notes in Russian with a shared corpus and shared relevance judgements. This statement is provisional: it depends on the search in section 6, which has not been run.

## 5. Hypotheses and falsification conditions

- **H1.** On short personal notes, the hybrid strategy (RRF) gives higher MRR than either sparse or dense alone. *Rejected* if the hybrid does not exceed the best single strategy on MRR on the labelled set.
- **H2.** A multilingual model gives higher Recall@K than an English-only model on Russian notes. *Rejected* if the English-only model's Recall@K is equal to or higher than the multilingual model's on the same queries.
- **H3.** Queries that paraphrase a note without shared keywords favour the dense strategy; queries with exact rare terms favour the sparse strategy. *Rejected* if, on the paraphrase subset, the dense strategy does not have higher Precision@K than the sparse one, or if, on the exact-term subset, the sparse strategy does not have higher Precision@K than the dense one.

Each condition must be checked with the statistical test defined in `verification_plan.md`, not only by comparing the means.

## 6. Search protocol (to be executed and recorded)

Run the following queries in Google Scholar and arXiv, and record the date, the number of results and the number of relevant close works found:

- "hybrid search" "personal notes" retrieval evaluation
- "personal knowledge base" retrieval BM25 dense
- "reciprocal rank fusion" evaluation sparse dense
- Russian language retrieval evaluation embeddings
- "note retrieval" OR "notes search" semantic keyword comparison

Any close work found must be added to the table with one sentence on how it differs from this project.

| Date | Query | Results | Close works found |
|---|---|---|---|
| to be filled | | | |
