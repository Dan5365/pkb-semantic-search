# Verification and Evaluation Plan

**Project:** PKB Semantic Search  
**Author:** Marat Yerkanat, SE-2407  
**Date:** 23 October 2026  
**Status:** Proposed protocol for implementation and evaluation

## 1. Purpose and Scope

This document defines how the PKB search module will be verified and how its retrieval strategies will be compared.

The evaluation covers:

1. Sparse retrieval using PostgreSQL full-text search.
2. Dense retrieval using embeddings and pgvector.
3. Hybrid retrieval using Reciprocal Rank Fusion (RRF).

The research question is which strategy places relevant Russian-language personal notes higher in the result list, and how its effectiveness changes across query types.

The main evaluation unit is a complete note identified by a stable `note_id`. Search quality is measured using Precision@K, Recall@K and Mean Reciprocal Rank (MRR).

Generated answers and chatbot quality are outside the evaluation scope. The evaluated output is a ranked list of notes.

The settings below are proposed experiment settings. The final configuration must be recorded before the final test run.

## 2. Retrieval Strategies

### 2.1 Sparse Retrieval: PostgreSQL Baseline

The keyword baseline uses PostgreSQL full-text search.

Proposed configuration:

- Text search configuration: `russian`.
- Searchable content: note title and body.
- Document representation: `tsvector` with positional information.
- Query terms: lexemes produced using the same Russian configuration.
- Query matching: OR between normalized query lexemes.
- Ranking: `ts_rank_cd` with normalization option `0` and default weights.
- Index: GIN on the search vector.
- Tie-breaking: ascending `note_id`.

OR matching allows a note to become a candidate when it contains at least one query lexeme. This policy is fixed for the evaluation.

PostgreSQL full-text search includes lexical normalization. Therefore, this baseline is more capable than exact substring matching, but it does not use embedding similarity.

The proposed search vector is:

```sql
to_tsvector(
    'russian',
    coalesce(title, '') || ' ' || coalesce(body, '')
)
```

For query construction, the application obtains normalized lexemes with:

```sql
SELECT tsvector_to_array(
    to_tsvector('russian', :query_text)
);
```

Each lexeme is quoted as a literal tsquery operand. The operands are joined with ` | ` and passed as the bound parameter `:lexical_query`. An empty lexeme list produces an empty result list.

The proposed retrieval query is:

```sql
WITH q AS (
    SELECT CAST(:lexical_query AS tsquery) AS value
)
SELECT
    n.note_id,
    ts_rank_cd(n.search_vector, q.value, 0) AS score
FROM notes AS n
CROSS JOIN q
WHERE n.search_vector @@ q.value
ORDER BY score DESC, n.note_id ASC
LIMIT :candidate_limit;
```

Table and column names will be mapped to the implemented database schema. Query values must be passed through bound parameters.

### 2.2 Dense Retrieval

Dense retrieval uses cosine distance between the query embedding and stored text embeddings.

Proposed configuration:

- Embedding baseline: `all-MiniLM-L6-v2`, as specified in the project README.
- Alternative pretrained models: compared on development queries.
- Similarity: cosine similarity, calculated as `1 - cosine_distance`.
- Production index: HNSW with `vector_cosine_ops`.
- Initial HNSW settings: `m = 16`, `ef_construction = 64`, `ef_search = 200`.
- Tie-breaking: ascending `note_id`.

The final model is selected using the development set. Its identifier, revision, vector dimension and inference settings are then frozen for the final strategy comparison.

Russian-language retrieval quality must be measured for each candidate model.

For long notes, text is split into overlapping chunks. The exact chunking algorithm, tokenizer, maximum size and overlap must be recorded. Chunks must fit the selected model's input limit without silent truncation.

A note's dense score is the highest similarity among its chunks. Each note appears only once in the ranked output.

### 2.3 Hybrid Retrieval

Hybrid retrieval combines the sparse and dense note rankings using equal-weight RRF:

```text
RRF(note) =
    sum of 1 / (60 + rank_source(note))
    over source lists containing the note
```

Rules:

- Ranks start at 1.
- The fusion constant is `60`.
- A missing note contributes zero for that source.
- Each source contributes once per note.
- Chunk results are grouped into notes before fusion.
- Final ordering is descending RRF score, then ascending `note_id`.

Dense and hybrid retrieval use the same embedding model and stored embeddings.

## 3. Evaluation Data

### 3.1 Starter Verification Corpus

The following synthetic corpus provides a small, fully specified fixture for development and verification.

The title and body together form the searchable text. These notes are examples created for this plan, rather than collected user data.

| Note ID | Title | Body |
|---|---|---|
| N01 | Локальное подключение PostgreSQL | Локальная база PKB доступна по адресу localhost на порту 5432. Имя базы — pkb. |
| N02 | Расширение pgvector | Для хранения векторов в PostgreSQL нужно включить расширение командой CREATE EXTENSION vector. После этого можно создавать столбцы типа vector. |
| N03 | Запуск сервисов | Все сервисы проекта запускаются командой docker compose up -d из папки с файлом Compose. |
| N04 | Диагностика контейнеров | Если контейнер падает, сначала проверь его журналы командой docker compose logs, указав имя сервиса. |
| N05 | Секреты в Git | Пароли и токены хранятся в локальном файле .env. Этот файл добавляется в .gitignore до первого коммита. |
| N06 | Работа через ветку | Для изменения проекта создай отдельную Git-ветку. После коммита и отправки ветки открой pull request для проверки изменений. |
| N07 | Точность поиска | Precision@K показывает долю релевантных заметок среди первых K позиций результата поиска. |
| N08 | Полнота поиска | Recall@K показывает, какая доля всех релевантных заметок найдена среди первых K позиций результата поиска. |
| N09 | Объединение ранжирований | RRF объединяет списки полнотекстового и векторного поиска. Для каждой заметки складываются вклады 1 / (60 + позиция) из двух списков. |
| N10 | Приближённый поиск HNSW | HNSW выполняет приближённый поиск соседей. Параметр ef_search задаёт размер списка кандидатов. Потери поиска проверяются сравнением с точным перебором. |
| N11 | Косинусное расстояние | В векторном поиске меньшее косинусное расстояние означает большую близость. Для ранжирования расстояние сортируется по возрастанию. |
| N12 | Разбиение длинных заметок | Длинная заметка разбивается на фрагменты. Перекрытие соседних фрагментов сохраняет контекст на границах. Каждый фрагмент хранит идентификатор исходной заметки. |
| N13 | Параметры поиска | Поисковый endpoint /search принимает текст запроса query, выбранную стратегию strategy и количество результатов k. |
| N14 | Запись к стоматологу | Приём у стоматолога назначен на пятницу в 16:00. Нужно прийти за десять минут до записи. |
| N15 | Бюджет на продукты | На продукты в следующем месяце запланировано 45000 тенге. Расходы записываются после каждой покупки. |
| N16 | Точный векторный поиск | Точный поиск рассчитывает расстояния до всех сохранённых векторов. Его результаты используются как контроль для проверки приближённого поиска HNSW. |

N13 is part of the synthetic fixture. The implemented API contract remains defined by the project's API documentation.

### 3.2 Labeled Starter Queries

The relevant-note column defines the positive labels for the starter corpus. All other notes in this corpus receive relevance label `0` for that query.

For queries containing several explicit information needs, a note is relevant when it directly answers one of those needs.

| Query ID | Query | Type | Relevant notes |
|---|---|---|---|
| Q01 | На каком порту работает локальная база PostgreSQL? | Keyword | N01 |
| Q02 | Как включить расширение для хранения векторов? | Paraphrase | N02 |
| Q03 | Как запустить все сервисы проекта одной командой? | Paraphrase | N03 |
| Q04 | Где посмотреть причину падения контейнера? | Paraphrase | N04 |
| Q05 | Как не отправить пароли и токены в репозиторий? | Paraphrase | N05 |
| Q06 | Как предложить изменение проекта через отдельную ветку? | Paraphrase | N06 |
| Q07 | Какая доля первых результатов действительно подходит к запросу? | Meaning-based | N07 |
| Q08 | Какая часть всех подходящих заметок попала в выдачу? | Meaning-based | N08 |
| Q09 | Как объединить результаты полнотекстового и векторного поиска? | Paraphrase | N09 |
| Q10 | Какой параметр HNSW управляет размером списка кандидатов? | Keyword | N10 |
| Q11 | Близкие векторы имеют меньшее или большее косинусное расстояние? | Paraphrase | N11 |
| Q12 | Как сохранить контекст на границе фрагментов длинной заметки? | Paraphrase | N12 |
| Q13 | Какие параметры принимает /search? | Identifier | N13 |
| Q14 | Когда у меня приём у зубного? | Synonym | N14 |
| Q15 | Сколько денег запланировано на еду в следующем месяце? | Synonym | N15 |
| Q16 | Как проверить потери HNSW относительно полного перебора? | Multiple relevant notes | N10, N16 |
| Q17 | Какие заметки объясняют точность и полноту поиска? | Multiple relevant notes | N07, N08 |
| Q18 | Как запустить сервисы и посмотреть их журналы? | Multiple information needs | N03, N04 |
| Q19 | Где описан точный поиск по всем векторам? | Keyword | N16 |
| Q20 | как запустит докер кампоз | Typo and transliteration | N03 |

These labels are proposed fixture labels. Team review is required before freezing the fixture.

This small corpus verifies the evaluation workflow. Its scores are insufficient for conclusions about real personal-note collections.

### 3.3 Main Evaluation Corpus

The proposed main dataset contains:

- At least 100 Russian-language personal notes.
- 60 answerable queries.
- 20 development queries.
- 40 held-out test queries.

Notes should cover several topics, such as study, technical work, project planning and everyday tasks.

The query set should include:

- Exact terms and identifiers.
- Synonyms and paraphrases.
- Short and informal wording.
- Abbreviations and spelling errors.
- Queries with multiple relevant notes.

Each note and query receives a stable identifier.

Paraphrases of the same information need belong to the same split. Development queries are used for model selection and parameter tuning. Test queries are used after the configuration is frozen.

The corpus remains shared across splits because the task evaluates retrieval over a fixed knowledge base.

### 3.4 Labeling Procedure

Binary relevance labels are used:

- `1`: the note directly answers the information need, or an explicit part of a multi-part query.
- `0`: the note does not provide the required information.

General topic similarity alone is insufficient.

Two team members independently assess every corpus note for each evaluation query. Disagreements are discussed and resolved before the labels are frozen.

Labels are assigned without showing strategy names or ranking positions.

Assessing the complete corpus gives Recall@K a known denominator.

Queries with no relevant notes are maintained separately as robustness cases. They are excluded from the main answerable-query averages.

## 4. Metrics

Let:

- `R(q)` be the set of all relevant note IDs for query `q`.
- `TopK(q)` be the first K unique returned note IDs.

The evaluation uses `K = 1, 5, 10`.

### 4.1 Precision@K

```text
Precision@K(q) = |TopK(q) ∩ R(q)| / K
```

If fewer than K notes are returned, the denominator remains K.

### 4.2 Recall@K

```text
Recall@K(q) = |TopK(q) ∩ R(q)| / |R(q)|
```

Every query in the main benchmark has at least one relevant note.

### 4.3 Mean Reciprocal Rank

For a complete ranking:

```text
RR(q) = 1 / position of the first relevant note
```

If the strategy returns no relevant note, `RR(q) = 0`.

```text
MRR = mean of RR(q) across evaluated queries
```

For output limited to the first 10 positions:

```text
RR@10(q) =
    1 / first relevant position, if that position is at most 10;
    0, otherwise

MRR@10 = mean of RR@10(q)
```

A metric calculated from a truncated result list must include its cutoff in its name.

### 4.4 Aggregation

Metrics are calculated per query and then macro-averaged across queries.

Results are reported:

- Over the complete held-out test set.
- By query type.
- As paired differences between strategies on the same queries.

Repeated timing runs do not increase the number of independent quality observations.

### 4.5 Hand-Calculated Verification Example

Suppose:

```text
Relevant notes: {N03, N04}
Returned ranking: [N05, N03, N01, N04, N02]
```

Expected values:

```text
Precision@5 = 2 / 5 = 0.4
Recall@5 = 2 / 2 = 1.0
RR = 1 / 2 = 0.5
```

This example is a metric test fixture, rather than an experimental result.

## 5. Evaluation Modes

Two modes are recorded separately.

### 5.1 Complete-Ranking Reference Mode

This offline mode supports unqualified MRR:

- Sparse retrieves all matching notes.
- Dense calculates exact distances over all chunks and ranks all notes after aggregation.
- Hybrid applies RRF to the complete sparse and dense note lists.
- Metrics: Precision@K, Recall@K and full MRR.

This mode provides a reference for retrieval effectiveness without approximate-index losses.

### 5.2 Indexed Application Mode

This mode evaluates the planned HNSW-based application:

- Sparse returns up to 100 unique note candidates.
- Dense retrieves up to 200 chunk candidates through HNSW, groups them by note and keeps up to 100 unique notes.
- Hybrid fuses those sparse and dense candidate lists.
- Metrics: Precision@K, Recall@K, MRR@10 and latency.

Actual candidate counts are recorded. Results from the two modes are kept in separate tables.

For an ANN-specific diagnostic, HNSW and exact vector search are compared at the same chunk candidate depth. Neighbor overlap is reported separately from relevance-based Recall@K.

An execution plan must confirm whether HNSW was actually used. A small corpus may cause PostgreSQL to select a sequential scan.

## 6. Measurement Conditions

The following information is recorded for every experiment:

| Category | Required information |
|---|---|
| Dataset | Corpus version, note count, chunk count, query split and relevance-label version |
| Source | Git commit used for the experiment |
| Database | PostgreSQL and pgvector versions |
| Model | Model identifier, revision, dimension, tokenizer and inference device |
| Text processing | Normalization, included fields, chunking algorithm, size and overlap |
| Sparse search | Language configuration, OR query policy, ranking function and normalization |
| Dense search | Distance function, candidate limits and HNSW settings |
| Hybrid search | RRF constant, source depths and tie-breaking |
| Environment | Operating system, CPU, GPU, RAM and container resource limits |
| Timing | Measurement boundary, warm-up, repetitions, cache policy and concurrency |
| Randomness | Seeds used for splitting, ordering and other randomized operations |

The same corpus snapshot, query text and relevance labels are used for all strategies.

Document embeddings and indexes are prepared before timed searches.

### Latency Protocol

Proposed conditions:

- One client with concurrency `1`.
- Model and database are warmed before measurement.
- Ten warm-up requests are excluded.
- Every test query is measured five times per strategy.
- Strategy and query order are randomized using a recorded seed.
- Query-embedding caching is disabled.
- Database buffers remain warm.

Timing starts when the search operation receives query text and ends when the ranked unique note IDs are ready.

Dense and hybrid timing includes query tokenization and embedding inference. It excludes document embedding, index construction, browser rendering and network transport.

Median and p95 latency are reported separately from retrieval quality.

## 7. Evaluation Procedure

1. Prepare and validate the corpus.
2. Assign stable note and query IDs.
3. Complete relevance judgments and resolve disagreements.
4. Freeze development and test query groups.
5. Verify the metrics using hand-calculated examples.
6. Compare embedding models on development queries.
7. Fix the model, chunking and retrieval parameters.
8. Build embeddings and indexes for the frozen corpus.
9. Run the complete-ranking reference evaluation.
10. Run the indexed application evaluation.
11. Run latency measurements under the recorded conditions.
12. Save ranked outputs, per-query metrics and aggregate results.
13. Analyze differences by query type and inspect failed queries.

Changes made after inspecting final test results require a new experiment version and must be disclosed.

## 8. Functional Verification

| Area | Verification |
|---|---|
| Data integrity | Note IDs are unique; query labels reference existing notes. |
| Corpus consistency | Every evaluated note has valid searchable text and embeddings. |
| Embeddings | Dimensions match the configured model; missing or invalid vectors are detected. |
| Sparse retrieval | Query normalization, OR matching and score ordering follow the fixed configuration. |
| Dense retrieval | Results are ordered by cosine distance and mapped to the correct notes. |
| Chunk aggregation | Multiple chunks from one note produce one note result. |
| RRF | Contributions, missing-source cases and tied scores match hand-calculated examples. |
| Metrics | Correct values for first relevant at rank 1, later relevance, no relevant result and multiple relevant notes. |
| Short result lists | Precision@K still uses K as its denominator. |
| Input handling | Empty and whitespace-only queries follow the documented API validation contract. |
| Updates | Editing a note updates its searchable text and embeddings. |
| Deletion | A deleted note disappears from all retrieval strategies. |
| Strategy comparison | The comparison endpoint uses the same query and corpus snapshot for every strategy. |
| Reproducibility | Re-running evaluation on identical ranked outputs produces identical metrics. |

Queries with no relevant notes are used to inspect irrelevant results and application behavior.

Dense top-K retrieval alone does not provide a reliable “no answer” decision. Any future rejection threshold must be selected on development data and evaluated separately.

## 9. Reporting and Completion Criteria

### Complete-Ranking Reference Results

| Strategy | P@1 | P@5 | P@10 | R@5 | R@10 | MRR |
|---|---|---|---|---|---|---|
| Sparse | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |
| Dense, exact | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |
| Hybrid, complete lists | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |

### Indexed Application Results

| Strategy | P@5 | R@5 | P@10 | R@10 | MRR@10 | p50 latency | p95 latency |
|---|---|---|---|---|---|---|---|
| Sparse | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |
| Dense, indexed configuration | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |
| Hybrid, bounded lists | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured | Not measured |

Recall@1 is also saved in the detailed evaluation output.

The evaluation is complete when:

- The corpus, query splits and relevance labels are versioned.
- All three retrieval strategies are evaluated on the same data.
- Required functional and metric checks pass.
- Experiment settings and execution plans are recorded.
- Ranked outputs and per-query metrics are available.
- Results and limitations are explained using measured evidence.

Any of the three strategies may achieve the highest score.

The synthetic starter fixture supports verification of the workflow. Research conclusions must be based on the larger held-out evaluation and must acknowledge its size, topic coverage and labeling limitations.

## 10. References

1. PostgreSQL 16 documentation. [Controlling Text Search](https://www.postgresql.org/docs/16/textsearch-controls.html).
2. pgvector. [Official documentation](https://github.com/pgvector/pgvector).
3. Cormack, G. V., Clarke, C. L. A., and Büttcher, S. (2009). [Reciprocal Rank Fusion Outperforms Condorcet and Individual Rank Learning Methods](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf).
