# Project Management

**Project:** PKB Semantic Search
**Owner:** Daniyal Adilbekuly, SE-2407
**Tracker:** YouTrack, project PKB
**Dates:** to be confirmed against the School's defense schedule.

## 1. Milestones

| ID | Milestone | Date | Done when |
|---|---|---|---|
| M1 | Proposal and scope | 19 Oct 2026 | `docs/project_proposal.md` committed and reviewed by the team |
| M2 | Requirements and architecture | 21 Oct 2026 | `docs/requirements.md` and diagrams with sources committed |
| M3 | Minimum requirements package | 23 Oct 2026 | All items of the minimum list are in the repository, with commit history from each member |
| M4 | Literature and user evidence | 23 Oct 2026 | Literature table with at least 12 sources (8 scientific); interview results separated into facts and assumptions |
| M5 | Labeled evaluation set | 20 Nov 2026 | 30–50 queries with relevant notes, labeling protocol documented |
| M6 | Core search module | 15 Dec 2026 | Sparse, dense and hybrid strategies behind the API; unit tests for RRF and metrics |
| M7 | Embedding service and comparison view | 31 Jan 2027 | Embedding service integrated; interface shows three strategies side by side |
| M8 | Experiments and analysis | 28 Feb 2027 | Metrics for all strategies and baseline; significance tests; error analysis |
| M9 | Thesis draft | 31 Mar 2027 | Full draft of the explanatory note reviewed by the supervisor |
| M10 | Defense preparation | April 2027 | Final version, presentation, reproducible run instructions |

## 2. Risks

| ID | Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R1 | Fewer than 3 interviews completed | Medium | High | Contact people this week; if fewer, report the actual number honestly | Tapishev |
| R2 | Labeling takes longer than planned or agreement is low | High | High | Start with 30 queries; second labeler on a subset; measure agreement | Marat |
| R3 | Schema or API changes block other members | Medium | High | Freeze the API contract before coding; changes via pull request and YouTrack | Daniyal |
| R4 | Embedding model is too heavy for development machines | Medium | Medium | Use the small model all-MiniLM-L6-v2; keep a documented fallback | Tapishev |
| R5 | Latency target (NFR-1) not met | Medium | Medium | Measure early on a small corpus; agree threshold with supervisor | Daniyal |
| R6 | A member is overloaded or inactive | Medium | High | Weekly status; reassign tasks in YouTrack | All |
| R7 | Small corpus limits the conclusions | High | Medium | State limits explicitly in the thesis; synthetic data only as a complement | Marat |
| R8 | Privacy of personal notes | Low | High | No real notes in the repository; consent per NFR-9; `.gitignore` for data | All |
| R9 | Supervisor feedback arrives too late | Medium | Medium | Meeting scheduled by 22 Oct; feedback logged in `docs/supervisor_feedback.md` | Daniyal |
| R10 | Commit history does not show individual contribution | Low | High | Every commit carries a task key from YouTrack | All |

## 3. Dependencies

- API contract (PKB-6) → frontend integration (Tapishev) and evaluation runs (Marat).
- Schema and vector index → embedding loading (Marat) and dense and hybrid search.
- Embedding model choice (PKB-13) → embedding service and chunking.
- Labeled evaluation set → metrics → comparison report.
- Baseline (keyword search in PostgreSQL) → verification plan (PKB-15).

## 4. Working rhythm

- Weekly meeting, 30 minutes, with a status update in YouTrack before it.
- Each task has one owner, a due date and a priority.
- Pull requests are reviewed by one other member before merge.

## 5. Change control

Scope changes are recorded as YouTrack issues and discussed with the supervisor before implementation. Milestones M5–M10 are planning estimates and are revised after the first measurements.
