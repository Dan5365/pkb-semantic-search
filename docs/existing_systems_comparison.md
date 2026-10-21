# Comparison of Existing Personal Knowledge Base Search Systems

**Project:** PKB Semantic Search  
**Author:** Marat Yerkanat, SE-2407  
**Date:** 20 October 2026  
**Method:** Documentation-based comparison

## 1. Purpose and Scope

This document compares four systems that support searching personal notes: Obsidian with Smart Connections, Notion, Logseq, and Khoj. The comparison identifies useful capabilities and informs the design of the PostgreSQL/pgvector semantic search module developed in the PKB project.

Eight criteria are considered: keyword search, semantic search, filtering, result presentation, storage and deployment, offline operation, supported content, and extensibility.

The comparison describes documented capabilities. It does not measure retrieval quality, latency, or usability. A feature marked “not established” was not found in the cited documentation page; this does not prove that the feature is unavailable.

## 2. Selected Configurations

- **Obsidian:** core Search plugin and the community Smart Connections plugin (version 4 according to the plugin repository). The free core plugin provides embedding-based semantic lookup and related-note discovery with a bundled local model. Advanced model, filter, and ranking options are described as Pro features. The plugin is source available under a license that restricts general-purpose competing Obsidian offerings; it is not described as open source. [O1, O3]
- **Notion:** workspace search and Notion AI Enterprise Search. Enterprise Search is available on Business and Enterprise plans and answers questions using workspace content, connected sources, and, by default, the web. [N1, N2]
- **Logseq:** the database (DB) graph version as described in the official DB documentation, which states that it describes functionality as of 28 April 2026. Logseq Sync and the iOS app are described there as invite-only. DB graphs must be distinguished from the older file graph version. [L1, L2]
- **Khoj:** semantic search and knowledge-base chat in two configurations: Khoj Cloud and self-hosted. [K1, K3, K4, K5]

These configurations combine built-in functionality, plugins, and service plans. Documented semantic capabilities are therefore attributed to the selected configuration rather than to the base application alone. [O3, N2, L1, K1]

## 3. Comparison Matrix

Each cell contains a documented claim followed by its source identifier (see References). “Not established” means the capability was not found in the cited page.

| Criterion | Obsidian + Smart Connections | Notion | Logseq DB | Khoj |
|---|---|---|---|---|
| **1. Keyword search** | Core Search supports text terms, quoted phrases, Boolean operators, and regular expressions. [O1] | Workspace search covers page titles and content; quoted phrases are supported for exact matches. [N1] | A search modal over nodes, queries, and filterable views are documented. Keyword syntax such as exact phrases or operators is not described in the cited pages. [L1, L2] | Required and excluded word filters are documented. A separate lexical ranking strategy is not established by the cited search documentation. [K1, K2] |
| **2. Semantic search** | Smart Connections provides semantic lookup (Lookup view) and related-note discovery (Connections view) using locally computed embeddings. Matching is by meaning, so a note containing the exact query text may not be returned. This capability belongs to the plugin. [O3] | Enterprise Search accepts natural-language questions and searches workspace content, connected apps, and by default the web. The cited documentation does not specify its ranking algorithm. [N2] | Built-in embedding-based retrieval is not established in the cited DB documentation. [L1, L2] | Uses embeddings of queries and document chunks, followed by cross-encoder reranking. [K1] |
| **3. Filtering and scope** | Core Search supports file, path, tag, content, and property filters. These are core-search capabilities and are not attributed to semantic lookup. [O1] | Workspace search includes filters such as creator and date. Enterprise Search can restrict its scope to selected sources, pages, or teamspaces. [N1, N2] | Views can filter nodes by properties, and query filters for tags and properties are documented. [L1, L2] | Supports word, file, and date filters. Date filters refer to dates contained in the indexed text. [K2] |
| **4. Results and source access** | Core Search shows matching content; Smart Connections displays related notes with similarity scores and expandable excerpts. [O1, O3] | Workspace search returns pages. Enterprise Search includes citations when answering from workspace or connected content. [N1, N2] | Search and query results refer to graph nodes (pages or blocks); nodes show linked references. [L1, L2] | Search retrieves relevant document chunks. Knowledge-base chat shows the reference notes used to generate a response. [K1, K3] |
| **5. Storage and deployment** | Notes are stored as local Markdown files in a vault. Smart Connections creates embeddings locally by default. [O2, O3] | Workspace content is stored in Notion’s cloud service and can be exported as PDF, HTML, Markdown, or CSV. [N5] | Desktop DB graphs use a local SQLite database file (`db.sqlite`) and an `assets/` directory. [L2] | Cloud and self-hosted deployment are available. The self-hosting guide refers to a PostgreSQL database for the server. [K4, K5] |
| **6. Offline operation** | Local notes and Smart Connections with its bundled local model are described as private and offline by default. [O2, O3] | Pages downloaded for offline use are available in the desktop and mobile apps, not in web browsers. Features that need connectivity, such as AI blocks, are unavailable offline. [N3] | Desktop DB graphs are stored locally. In the browser, multiple tabs stay in sync even when offline, without a sync server. [L2] | Self-hosting on a personal computer can run without an internet connection when the required models run locally. External model APIs require connectivity. [K1, K4, K5] |
| **7. Content formats** | The note format is Markdown. Core Search searches the content of notes and canvases; this does not imply full-text indexing of every attachment format. [O1, O2] | Direct imports include plain text, Markdown, Word, CSV, HTML, PDF, and ZIP, which are converted into Notion content. [N4] | DB graphs can import Markdown-based file graphs. Markdown is the only supported note format and Org-mode files are no longer supported. This does not mean notes are stored as individual Markdown files. [L1, L2] | Documented sources include Markdown, Org-mode, PDF, plain text, Word, GitHub repositories, and Notion pages. [K5] |
| **8. Extensibility** | The free core plugin works with a bundled local model without configuration. Configurable ranking, advanced filters and models, and API-based models are described as Pro features. [O3] | AI Connectors extend the sources available to Enterprise Search. [N2] | Documentation describes plugins that support DB graphs, an optional MCP server, a command-line interface, and scripting. [L1] | Self-hosted search allows configuration of embedding models and inference providers; the architecture is described as modular. [K1, K5] |

## 4. Implications for PKB Semantic Search

The reviewed systems demonstrate that semantic retrieval and natural-language access to notes already exist. Smart Connections and Khoj document embedding-based search, and Notion documents question answering over workspace content. The PKB project therefore does not claim that semantic search for personal notes is a new concept. [O3, N2, K1]

Within the cited documentation, none of the reviewed systems describes a controlled comparison of sparse, dense, and hybrid retrieval on a shared corpus, or an evaluation against a keyword baseline. Support for Russian or Kazakh search is also not established in the cited pages for any of the reviewed systems. The Khoj documentation indicates that multilingual search may require configuring a different embedding model, without stating which languages are supported. [K1]

The Smart Connections documentation notes that semantic queries behave differently from plain text search and may omit a note that contains the exact query text. This supports evaluating a hybrid strategy that combines both signals. [O3]

The project’s research contribution is a controlled comparison of three retrieval strategies within one PostgreSQL-based implementation:

1. **Sparse retrieval:** PostgreSQL full-text search. The exact ranking function is fixed in the verification plan.
2. **Dense retrieval:** embedding similarity search through pgvector.
3. **Hybrid retrieval:** combination of sparse and dense rankings using Reciprocal Rank Fusion.

All strategies will use the same corpus, relevance judgments, and evaluation procedure. This allows differences in retrieval effectiveness to be investigated under controlled conditions. The use of PostgreSQL as a database is not itself novel: the Khoj self-hosting guide refers to a PostgreSQL database. [K4]

The comparison also motivates the following design decisions:

- Return note identifiers and relevant excerpts so users can inspect the original material.
- Keep retrieval strategies independently selectable.
- Record model, indexing, and ranking settings to make experiments reproducible. The Khoj documentation states that changing the embedding model requires re-indexing all documents. [K1]
- Evaluate semantic and hybrid search against a keyword baseline before claiming an improvement.
- Keep generated answers outside the initial evaluation scope: the primary output is a ranked list of notes.

No conclusion about which reviewed product has the best search quality can be drawn from documentation alone.

## 5. References

All sources were consulted on 20 October 2026. Logseq documentation was read from the `master` branch; the page states that it describes functionality as of 28 April 2026.

- **O1.** Obsidian. [Search](https://help.obsidian.md/plugins/search).
- **O2.** Obsidian. [How Obsidian stores data](https://help.obsidian.md/data-storage).
- **O3.** Smart Connections. [Official plugin repository and documentation](https://github.com/brianpetro/obsidian-smart-connections).
- **N1.** Notion. [Search](https://www.notion.com/help/search).
- **N2.** Notion. [Enterprise Search](https://www.notion.com/help/enterprise-search).
- **N3.** Notion. [Use pages offline](https://www.notion.com/help/use-pages-offline).
- **N4.** Notion. [Import data into Notion](https://www.notion.com/help/import-data-into-notion).
- **N5.** Notion. [Export your content](https://www.notion.com/help/export-your-content).
- **L1.** Logseq. [Database version documentation](https://github.com/logseq/docs/blob/master/db-version.md).
- **L2.** Logseq. [Changes in the database version](https://github.com/logseq/docs/blob/master/db-version-changes.md).
- **K1.** Khoj. [Search](https://docs.khoj.dev/features/search/).
- **K2.** Khoj. [Query filters](https://docs.khoj.dev/miscellaneous/query-filters/).
- **K3.** Khoj. [Chat](https://docs.khoj.dev/features/chat/).
- **K4.** Khoj. [Self-hosting setup](https://docs.khoj.dev/get-started/setup/).
- **K5.** Khoj. [Features overview](https://docs.khoj.dev/features/all-features/).
