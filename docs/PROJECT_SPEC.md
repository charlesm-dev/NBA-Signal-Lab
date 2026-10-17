# NBA Signal Lab — Project Specification

This is the project charter (roadmap Stage 1). It defines what NBA Signal Lab is, what the MVP must do, and what it explicitly will not do. Architecture lives in [ARCHITECTURE.md](ARCHITECTURE.md); staged delivery in [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md); rationale for decisions in [DECISIONS.md](DECISIONS.md).

**Status labels used throughout:** **FIXED** (not changed without explicit agreement), **AGREED** (target, details may evolve), **PROPOSED** (recommended, not yet decided), **CONDITIONAL** (only if a measured need appears), **UNRESOLVED** (needs a decision or evidence).

## 1. Vision

NBA Signal Lab is a backend-first **basketball event-data ingestion, normalization, historical analysis and game-state intelligence platform**. It consumes usable NBA data (schedules, teams, players, games, box scores, play-by-play), preserves canonical identities and provenance, reconstructs basketball state (especially possessions), derives statistics and historical comparisons the provider does not expose directly, and serves them through analytical APIs.

Its distinguishing value is the engineered pipeline:

**source data → normalized event history → deterministic derived basketball state → queryable historical intelligence → (later, optionally) near-live replay experiences**

The NBA domain is essential, not cosmetic.

### Purpose

The primary objective is a substantial, credible, **portfolio-quality backend project** that the developer builds personally and can explain in engineering interviews. It is not a commercial launch and must not be presented as equivalent to professional production experience.

### Intended users — UNRESOLVED

NBA enthusiasts and analysts who want richer historical and game-state analysis; potentially developers consuming the APIs. Exact persona, UX and API access model are not validated. No monetization.

## 2. Non-goals

NBA Signal Lab is **not**:

- a basic scores app, standings dashboard, or UI wrapped around a third-party NBA API
- a betting or fantasy-sports product or clone
- a frontend-heavy product (API docs and Django admin are the initial "UI")
- a licensed live-data product — no claim of real-time NBA feed access; historical replay is the substitute
- an ML/prediction project — predictions are optional and late, never at the expense of correctness
- a microservices showcase — one monorepo, two framework processes, nothing more without justification
- a keyword checklist — no technology is added solely for résumé coverage

## 3. MVP functional requirements (through milestone M2)

1. **Acquire a usable, permitted NBA historical dataset.** Evaluate legally appropriate sources; obtain at least one real complete game (with play-by-play if supported), then a meaningful historical slice (initially one season). Capture provenance: external IDs, ingestion timestamps, parser versions.
2. **Canonical identity and domain.** Seasons, teams, players, roster membership over time, games, box-score stats, ordered play-by-play. Provider IDs map to stable internal IDs.
3. **Repeatable ingestion.** Import teams → players → schedule/games → box scores → events; validate; upsert or version; prevent duplicates; surface failed/partial batches.
4. **Game/event access (FastAPI).** Team, player, game, game-events and player-game-history endpoints with pagination, versioned responses, consistent errors and OpenAPI docs.
5. **Derived NBA intelligence.** Normalized event taxonomy; deterministic possession reconstruction with explicit handling of rebounds, fouls, free throws, substitutions, period changes and data errors; score reconciliation validated against known games.
6. **Historical analytical queries.** Historical game-state similarity (e.g. "Q4, trailing by 10, 5:00 left"), rolling player/team form, scoring runs/comebacks, game comparisons. Exact formulas and filters are **UNRESOLVED** and decided at Stages 7–8.
7. **Product workflows (Django).** Login, workspaces, permissions, saved analysis configurations, watchlists if useful, admin.
8. **Reproducibility.** Source fixtures, migrations, repeatable rebuild of derived state, automated tests, baseline latency/query metrics, developer documentation.

### Advanced — CONDITIONAL, after measured need (M3+)

- Background backfill/recomputation jobs with durable retries and visible status (Celery/RabbitMQ candidate).
- Redis cache for measured hot/expensive reads; correctness never depends on the cache.
- Independent event consumers (Kafka) only if real fan-out/replay needs justify it.
- Historical replay at real/accelerated speed over WebSockets, with reconnect snapshots and lag measurement.
- Alerts/watchlists; optional model inference.
- Scaled backfills, correction propagation, object storage of permitted raw payloads, observability, cloud staging, load/failure testing.

## 4. Example workflows

- **Analyst:** sign in (Django) → choose a historical game state → FastAPI queries canonical/derived data → see comparable situations → save the query to a workspace (Django).
- **Data maintainer:** fetch a permitted source game → record ingestion batch and raw metadata → validate and normalize → write canonical entities/events → reconstruct possessions → expose via versioned API; reprocess on source correction.
- **Replay subscriber (advanced):** select archived game → replay ordered events → consumers update projections → WebSocket pushes state changes → reconnecting client fetches an authoritative snapshot.

## 5. Data constraints

No NBA provider, license, rate limit, historical depth or live entitlement has been verified yet. Stage 2 (data feasibility) is a **blocking gate**: no schema or framework work begins until a source is evaluated and one real complete game is obtained under acceptable terms. Findings will be recorded in `docs/DATA_SOURCES.md` (created in Stage 2).

**Public demo intent (confirmed by developer, 2026-10-16):** the project should eventually have a public demo **if data licensing allows it**. Source evaluation must therefore check redistribution and public-display rights, not only personal-use access. If no source permits public display, the fallback is a public demo over permitted/derived data only, or a private demo with a recorded walkthrough.

## 6. Milestones and success criteria

| Milestone | Stage | Success criteria |
|---|---|---|
| **M1** vertical slice | 6 | Permitted real game ingested into canonical PostgreSQL; tested FastAPI responses; first baseline metrics. Tag `v0.1.0`. |
| **M2** functional MVP | 8 | Validated possession/game-state analysis over meaningful history; Django product workflows; repeatable ingestion/rebuild; documented API; automated tests. Tag `v0.2.0-mvp`. |
| **M3** advanced backend | 13 | *Where justified:* measured async processing and caching; independent consumers/replay; proven duplicate/retry/recovery behavior. |
| **M4** production-like | 19 | Hardened auth/security, failure handling, Docker/CI/telemetry, reproducible secure cloud staging, backups and rollback. |
| **M5** portfolio-ready | 21 | Diagrams, ADRs, reproducible benchmarks with **actual** before/after results, explainable domain correctness and tradeoffs. |

### Honest claims

Résumé bullets and docs use measured numbers only. The project does **not** demonstrate: professional production tenure, real NBA-scale traffic, large-team or on-call ownership, real customer incidents, enterprise compliance, or licensed live-data access. Portfolio-scale simulation is labeled as such.

## 7. Constraints

- **Stack (FIXED):** Python, Django (+ DRF), FastAPI (+ Pydantic), PostgreSQL.
- **Learning-first (FIXED):** the developer implements the core components personally, with mentoring.
- **Schedule (developer-stated, 2026-10-16):** goal of ~2 months at roughly 1 hour/day (~50–60 hours), no hard deadline. The developer is new to Python and new to Django/FastAPI. **Planning implication:** M1 is a realistic 2-month target; M2 is a stretch; M3+ will extend beyond two months. Stages are dependency gates, not calendar promises. Scope will be cut by deferring CONDITIONAL work, never by skipping tests or validation.
- **Budget:** UNRESOLVED. Cloud spend is decided at Stage 19.

## 8. Open questions

| Question | Needed by |
|---|---|
| Which data source(s), under which license; does play-by-play include substitutions and reliable ordering? | Stage 2 |
| Raw payload storage and correction versioning approach | Stage 3 |
| Persistence strategy for NBA-owned schema (see ADR-007) | Stage 3 |
| Workspace role set; identity handoff Django → FastAPI | Stage 5 / 15 |
| Exact possession rules and game-state filters; which rolling stats are mandatory | Stages 7–8 |
| Target persona and whether any UI beyond API docs/admin is needed | Stage 8 |
| Cloud budget and public-demo topology | Stage 19 |
