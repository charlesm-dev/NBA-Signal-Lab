# NBA Signal Lab — Development Roadmap

**Status:** planning document; current progress is tracked in [DEVELOPMENT_LOG.md](DEVELOPMENT_LOG.md), not here. **Rule:** build a correct vertical slice before adding infrastructure. All stage deliverables are proposed, not completed. **Required stack:** Python, Django, FastAPI, PostgreSQL. Other components are conditional on measured need.

## Amendments (2026-10-16)

Agreed clarifications, recorded in [DECISIONS.md](DECISIONS.md). Stage numbers are unchanged so references stay valid.

- **Execution order (ADR-012):** 1 → 2 → 3 → 4 → **6 (M1)** → **5** → 7 → 8 (M2) → 9 onward. Django (Stage 5) no longer gates M1.
- **Ingestion (ADR-010):** a Python package + CLI on the NBA data-plane side, not FastAPI request handlers.
- **Early tooling (ADR-011):** Postgres-only Docker Compose from Stage 3; basic CI (lint + tests) once code exists. Stages 17–18 mature these.
- **Document names (ADR-013):** the charter is `docs/PROJECT_SPEC.md`; ADRs live in `docs/DECISIONS.md`; the feasibility report is `docs/DATA_SOURCES.md`.

## Milestones

- **M1 — Vertical slice (stage 6):** real, permitted NBA data → provider adapter → canonical PostgreSQL records → tested FastAPI game/team/player endpoints; one complete game's events accessible. Tag `v0.1.0`.
- **M2 — Functional MVP (stage 8):** reproducible ingestion, play-by-play normalization, validated possession reconstruction, historical game-state and rolling analytics, Django users/workspaces/saved analyses; tests and baseline metrics. Tag `v0.2.0-mvp`.
- **M3 — Advanced backend (stage 13, conditional):** asynchronous backfills, measured cache improvements, reliable independent event consumers if genuinely justified; retry/replay evidence. Tag `v0.3.0` if reached.
- **M4 — Production-like (stage 19):** hardened security, failure tests, containers, CI, telemetry, reproducible cloud staging/deployment and recovery procedures. Release tag only after verification.
- **M5 — Portfolio-ready (stage 21):** architecture diagrams, ADRs, reproducible benchmarks and real before/after results, comprehensive README, demonstrable failures/recovery, clear interview narratives.

## Stage-by-stage execution

Each stage has **required** deliverables; items marked **later** must not block progression. Each PR should link an issue, contain tests appropriate to its change, and update relevant docs.

### 1. Charter and repository
- **Objective / why now:** constrain the product before design proliferates.
- **Build / execute:** create repository, `docs/PROJECT_SPEC.md` (charter), README stub, `docs/DECISIONS.md` (ADRs), GitHub Issues for next three stages, branches/PR workflow. Charter states target users, core historical NBA analysis, MVP scope and non-goals (no betting/fantasy clone, frontend-heavy product, immediate live licensed feed, ML, microservices).
- **Tech:** Git, GitHub, Markdown. **Reinforce:** general API/project concepts. **Learn:** ADRs, scoping, reviews.
- **Decide:** precise first differentiating analytic (recommended possession reconstruction + historical game-state comparison); users/personas remain unvalidated.
- **Done / tests:** charter committed; milestone definitions and exclusions documented; issues exist. No code tests.
- **Pitfall / interview:** overplanning; explain incremental architecture and scope control.

### 2. Data feasibility spike — blocking gate
- **Objective / why:** validate real data and permissions *before* final schema.
- **Build / execute:** disposable Python HTTP/data script; inspect teams, players, seasons, schedule, one complete game's box score and play-by-play; save legally reusable, sanitized fixtures; write `docs/DATA_SOURCES.md` comparing official/public, third-party, historical/open and paid sources, availability, pricing, IDs, history, rate limits, update frequency, corrections, live delay, permitted uses, attribution and redistribution limits. Verify substitutions, event order and timestamps if possession reconstruction is intended.
- **Tech:** Python, HTTPX, JSON, candidate dataset/provider. **Reinforce:** REST, JSON, HTTP/security basics. **Learn:** provider contracts, provenance, licensing and rate limits.
- **Decide:** provider and fallback; whether historical play-by-play supports chosen analytics; live/replay feasibility. **Done:** one real complete game's raw data retrieved under acceptable terms, stable IDs mapped conceptually, documented field gaps and license; otherwise reduce scope/revisit source, don't proceed on assumptions.
- **Tests:** fixture validation / smoke scripts; no live calls in unit tests. **Pitfall:** treating public access as permission to redistribute; explain source evaluation.

### 3. Canonical NBA schema
- **Objective / why:** isolate provider IDs and enforce domain invariants.
- **Build / execute:** ERD and migrations for `Team`, `Player`, `Season`, `RosterMembership`, `Game`, `ProviderEntityMapping`, `IngestionBatch`, `RawGamePayload` metadata; add `GameEvent`, `PlayerGameStats`, `TeamGameStats` only where source supports. Seed sample games. Separate Django control-owned tables from analytics-owned tables/schemas and migration ownership.
- **Tech:** PostgreSQL (local via Postgres-only Docker Compose, ADR-011), Python, SQLAlchemy/Alembic **proposed** for NBA schema (ADR-007). **Reinforce:** JSON/REST concepts. **Learn:** SQL, normalization, keys, constraints, migrations, indexes, provenance.
- **Decide:** canonical key types, seasons, roster time validity, raw payload storage, source correction versioning, ORM/migration ownership. **Done:** clean migration works; FK/unique constraints tested; IDs map without collisions; ERD saved.
- **Tests:** database constraints, migrations, mapping uniqueness. **Pitfall:** leaking provider IDs into every table; explain data modeling tradeoffs.

### 4. Synchronous provider adapter and ingestion
- **Objective / why:** correct ingestion before queues.
- **Build / execute:** ingestion package + CLI (ADR-010) with a provider interface for teams/players/schedule/game/box score/play-by-play; validation/normalization; transactional upserts; ingestion status/errors; import sample data in dependency order; reimport same game five times.
- **Tech:** Python, HTTP client, PostgreSQL, chosen ORM. **Reinforce:** REST/JSON. **Learn:** adapter pattern, idempotency, transactions, correction handling.
- **Decide:** upsert vs versions, batch transaction boundaries, malformed payload policy. **Done:** no duplicate canonical game/events, failed batches visible, adapter isolated. **Tests:** parser unit, DB integration, duplicate import, malformed response and rollback. **Pitfall:** assuming HTTP 200 means valid data; explain replayable ingestion.

### 5. Django control plane (built after Stage 6 / M1 — ADR-012)
- **Objective / why:** establish user-facing product ownership after useful NBA data exists.
- **Build / execute:** Django/DRF models for `User`, `Workspace`, `WorkspaceMembership`, `SavedAnalysis`, `Watchlist`, `DataSourceConfiguration` where needed, `AuditEvent`; Django admin; minimal authentication and object-level workspace permissions; simple saved-analysis endpoints.
- **Tech:** Django, DRF, PostgreSQL. **Reinforce:** authentication, cookies/tokens, REST, server security. **Learn:** Django ORM/admin/migrations/permissions.
- **Decide:** workspace role set, separate schema/DB ownership, eventual identity handoff to FastAPI. **Done:** user login and workspace/saved analysis operations; cross-workspace access rejected. **Tests:** auth, object-level authorization, DRF API. **Pitfall:** mixing analytical computation into Django; explain control-plane rationale.

### 6. FastAPI NBA data API — M1
- **Objective / why:** expose a complete real-data vertical slice.
- **Build / execute:** Pydantic response contracts; `GET /v1/teams/{id}`, `/players/{id}`, `/games/{id}`, `/games/{id}/events`, `/players/{id}/games`; pagination, errors, OpenAPI docs; record baseline query counts and p50/p95 latency.
- **Tech:** FastAPI, Pydantic, PostgreSQL, NBA ORM. **Reinforce:** REST/JSON. **Learn:** DI, API versioning, async-vs-sync, response schemas.
- **Boundary:** Django owns users/workspaces/saved analyses; FastAPI owns provider-facing NBA ingestion and analytics APIs. Initially both in one monorepo with separate processes and clearly owned DB tables. Prefer explicit API contracts for cross-boundary calls; do not let both ORMs write the same tables. Identity propagation is a later decision.
- **Decide:** sync vs async DB, API pagination and public IDs. **Done:** one complete real game, events, team/player lookups, docs, API tests and baseline. Tag M1. **Tests:** schema/API, pagination, 404, integration. **Pitfall:** ORM leakage, async for its own sake; explain framework boundary.

### 7. Play-by-play interpretation
- **Objective / why:** produce proprietary derived analysis rather than proxy source APIs.
- **Build / execute:** canonical event taxonomy (shots, rebounds, turnovers, fouls, free throws, substitutions, periods), deterministic possession state machine; `Possession` linked to ordered events; possession counts, points, scoring runs, offensive/defensive metrics as data supports.
- **Tech:** Python, PostgreSQL, FastAPI. **Reinforce:** JSON. **Learn:** event-state machines, domain invariants, complex basketball edge cases.
- **Decide:** possession boundary rules, offensive rebounds, technical FTs, corrections, event uncertainty. **Done:** manually inspected games, repeatable derived output, scoring reconciliation with documented exceptions; no silent unknown-event drops. **Tests:** edge-case unit tests, golden games, regression suite. **Pitfall:** claiming accuracy without validation; explain state derivation.

### 8. Historical analysis and MVP — M2
- **Objective / why:** useful product workflows before infrastructure optimization.
- **Build / execute:** load a meaningful permitted history (start with one season); game-state historical comparison, rolling player/team form, scoring-run/comeback queries, saved analysis through Django; measure DB row counts, query counts, p50/p95 latency.
- **Tech:** Django, FastAPI, PostgreSQL/advanced SQL. **Reinforce:** REST/auth. **Learn:** window functions, aggregation, precompute-vs-on-demand.
- **Decide:** precise game-state filters and analytical definitions, materialization criteria. **Done:** repeatable history ingestion, validated possessions, usable comparisons, saved analyses, automated tests and baselines. Tag MVP. **Tests:** analytics correctness, API and regression, small performance baseline. **Pitfall:** adding predictions/ML before reliable stats; explain derivation tradeoffs.

### 9. Corrections, reliability and replay
- **Objective / why:** make source-of-truth and rebuild paths explicit.
- **Build / execute:** ingest states (`RECEIVED`, `NORMALIZED`, `DERIVED`, `FAILED`), parser/algorithm versions, reprocess/rebuild-game command, source correction tracking, partial-failure recovery.
- **Tech:** Python, PostgreSQL; object storage later if raw volume warrants. **Learn:** replay, idempotency, versioning, transactional boundaries.
- **Decide:** immutable raw snapshots vs updates, derivation invalidation and atomic replacement. **Done:** replay stable; correction reflected; partial imports detected; parser version recorded. **Tests:** duplicate, correction, crash, replay. **Pitfall:** losing original data; explain source vs projection.

### 10. SQL profiling before cache
- **Objective / why:** diagnose slow queries before hiding them behind Redis.
- **Build / execute:** grow representative data, `EXPLAIN (ANALYZE, BUFFERS)`, inspect N+1, composite indexes, query shapes, optional materialized views; benchmark three slow useful endpoints before/after.
- **Tech:** PostgreSQL, SQLAlchemy/Django ORM. **Learn:** planner, index tradeoffs, N+1, query optimization.
- **Decide:** indexes supported by actual predicates and cardinality. **Done:** query plans and reproducible before/after documented; correctness unchanged. **Tests:** SQL result regression and benchmarks. **Pitfall:** indexing every column; explain actual bottleneck.

### 11. Long-running jobs
- **Objective / why:** historical backfills/rebuilds no longer fit synchronous request path.
- **Build / execute:** first migrate one measured long-running operation to Celery/RabbitMQ; add job status, retry/backoff/timeouts, worker metrics; expand only as justified.
- **Tech:** Celery, RabbitMQ, PostgreSQL. **Learn:** at-least-once tasks, ACKs, idempotent workers, concurrency.
- **Decide:** retryable vs permanent errors, concurrency and timeouts. **Done:** API no longer blocks, crashes recover, duplicates safe, throughput/queue wait measured. **Tests:** worker integration, retry/crash, duplicate tasks. **Pitfall:** assuming exactly-once; explain queues vs streams.

### 12. Measured hot-read caching
- **Objective / why:** address repeated expensive analytics or current-game reads.
- **Build / execute:** one targeted Redis cache-aside path; explicit keys/TTL/invalidation and fallback; optional rate limiting. Compare baseline p95, DB query volume and cache-hit ratio.
- **Tech:** Redis, FastAPI. **Reinforce:** caching, HTTP caching. **Learn:** invalidation, staleness, Redis ops.
- **Decide:** freshness guarantees, correction invalidation. **Done:** measured benefit; Redis outage doesn't break correctness; tests pass. **Tests:** hit/miss/invalidation/outage. **Pitfall:** caching everything; explain consistency tradeoff.

### 13. Independent event consumers — conditional M3
- **Objective / why:** decouple multiple genuine downstream consumers of normalized/derived NBA changes.
- **Build / execute:** only if needed, Kafka events such as `GameEventNormalized`, `GameCompleted`, `PossessionsRebuilt`, `StatsUpdated`; begin with two independent consumers (e.g. projections and alerts), keyed by `game_id`; durable publication mechanism (outbox or equivalent) when DB writes and event publishing must agree; lag/DLQ/replay metrics.
- **Tech:** Kafka or compatible broker, Python, PostgreSQL. **Learn:** partitions, offsets, ordering scope, schema evolution, at-least-once, outbox.
- **Decide:** whether Celery/RabbitMQ already suffices (if so defer Kafka), event schema, retention, poison handling. **Done if built:** duplicate/restart/replay tested, lag measurable, no silent lost DB-to-broker event. **Tests:** consumer crash, duplicates, out-of-order, publish failure. **Pitfall:** using Kafka solely for résumé; explain job queue vs event log.

### 14. Historical near-live replay and WebSockets — conditional
- **Objective / why:** demonstrate live-style NBA behavior without assuming licensed live feed.
- **Build / execute:** deterministic replay of a historical game on adjustable clock, push derived state via FastAPI WebSocket, reconnect snapshot/cursor handling, delivery-latency measurement; Kafka only if already justified.
- **Tech:** FastAPI WebSockets, existing queue/stream as applicable. **Learn:** fan-out, reconnect, backpressure.
- **Decide:** state snapshots vs event pushes, heartbeat and staleness. **Done:** replay reaches known final state; reconnect safe; latency recorded. **Tests:** deterministic replay, disconnect, multi-client. **Pitfall:** WebSocket messages as source of truth; explain delivery vs durability.

### 15. Mature cross-service identity/security
- **Objective / why:** prepare public deployment without conflating authentication with authorization.
- **Build / execute:** Django identity authority; FastAPI verifies signed short-lived tokens or equivalent; workspace/object-level RBAC, scoped hashed API keys, expiry/revocation approach, rate limits, audit trails, threat model.
- **Tech:** Django, FastAPI, appropriate JWT/OAuth tooling. **Reinforce:** JWT, OAuth, bcrypt/scrypt, HTTPS/TLS, CORS/CSP, OWASP/server security. **Learn:** RBAC, key rotation, tenant scoping.
- **Decide:** claims vs DB authorization lookup, refresh/revocation, key scopes. **Done:** cross-workspace/expired/tampered-token tests; no plaintext stored API keys. **Tests:** authn/authz/security/rate limit. **Pitfall:** trusting role claims forever; explain identity boundary.

### 16. Failure-engineering test suite
- **Objective / why:** verify resilience of accumulated dependencies.
- **Build / execute:** failure matrix for provider outage/429, malformed source, DB/Redis/broker outage, worker death, duplicate/out-of-order/poison events, WebSocket loss, partial ingest; documented retry/degrade/alert decisions.
- **Tech:** pytest, Testcontainers or Docker test stack. **Learn:** failure domains, bounded backoff, circuit breaking when justified.
- **Decide:** acceptable data loss/staleness per flow. **Done:** critical recovery tests automated; no unbounded retries. **Tests:** unit/integration/contract/failure/concurrency. **Pitfall:** mock-only testing; explain recovery guarantees.

### 17. Reproducible containers
- **Objective / why:** make multi-process system start predictably.
- **Build / execute:** Dockerfiles for Django/FastAPI/worker/consumers as present; Compose with actual dependencies; health checks, migrations, non-root runtime, env config.
- **Tech:** Docker, Compose. **Learn:** image layers, volumes, networking, readiness. **Decide:** process/image boundaries and local vs prod config. **Done:** fresh clone can start and run smoke tests; no embedded secrets. **Tests:** clean startup/migration. **Pitfall:** one giant image or premature Kubernetes; explain containers vs orchestration.

### 18. CI and observability
- **Objective / why:** safe change flow and traceable operations.
- **Build / execute:** GitHub Actions lint/type/unit/integration/migration/build/scan; structured logs and correlation IDs; OpenTelemetry traces, Prometheus/Grafana metrics for API latency, provider errors, ingestion throughput, queue delay, consumer lag, cache hit, DB latency, WebSockets as present.
- **Tech:** GitHub Actions, OpenTelemetry, Prometheus/Grafana. **Learn:** SLIs, traces vs logs vs metrics. **Decide:** alert-worthy signals and data retention. **Done:** PR checks; cross-component trace; actionable dashboards. **Tests:** CI and telemetry smoke. **Pitfall:** dashboards without operational questions; explain signal selection.

### 19. Cloud deployment — M4
- **Objective / why:** operate beyond laptop.
- **Build / execute:** AWS staging: ECS/Fargate for Django/FastAPI/workers, RDS PostgreSQL, S3 for raw archives if needed, managed Redis if used, Secrets Manager, IAM, TLS/ALB, backup/restore and migration/rollback procedure; Terraform for repeatable infra. Broker selection must fit cost and actual workload; do not assume managed Kafka affordable.
- **Tech:** AWS, Docker, Terraform, GitHub Actions. **Reinforce:** HTTPS/TLS/server security. **Learn:** IAM, cloud networking, managed infra, IaC. **Decide:** budget, staging vs public demo, service cost, deployment topology. **Done:** secure HTTPS staging, health checks, backups, repeatable deploy, smoke and rollback rehearsals. **Tests:** deployed smoke, migration, recovery. **Pitfall:** public DB/excess IAM; explain cost/security tradeoffs.

### 20. Performance and scale experiments
- **Objective / why:** replace claims of scalability with measured evidence.
- **Build / execute:** repeatable API RPS tests (e.g. 50/100/250/500 if budget permits), backfill volume tests, event lag, replay/WebSocket fan-out; capture p50/p95/p99, error rate, DB/CPU, worker throughput. For each optimization record baseline → hypothesis → one change → retest.
- **Tech:** k6 or Locust, PostgreSQL profiling, metrics stack. **Learn:** load vs stress vs soak, bottlenecks, backpressure, scaling.
- **Decide:** first real bottleneck, vertical vs horizontal scaling. **Done:** at least three genuine documented before/after engineering improvements, with honest test conditions; do not invent numbers. **Tests:** repeatable load/regression. **Pitfall:** peak RPS without error/latency; explain bottleneck and fix.

### 21. Portfolio/interview packaging — M5
- **Objective / why:** make architecture and measured engineering legible.
- **Build / execute:** complete README, setup and data-permission caveats, API docs, ERD, deployment and event diagrams, ADRs, failure matrix, benchmarks, screenshots if useful, résumé bullets using *actual* results; tag portfolio release.
- **Tech:** GitHub, Markdown, diagram tool. **Learn:** technical communication. **Decide:** strongest three evidence-backed stories. **Done:** fresh-clone instructions work; all docs match implementation; explain each dependency, its alternative and measured reason. **Tests:** final end-to-end smoke. **Pitfall:** buzzword stack listing; explain engineering outcomes.

## Master table

| Stage | Goal | Technology | Deliverable | Dependencies |
|---|---|---|---|---|
| 1 | Scope | Git/GitHub | Charter/repo | — |
| 2 | Verify data | Python/HTTP | Source report + real fixture | 1 |
| 3 | Domain | PostgreSQL | ERD/migrations | 2 |
| 4 | Ingestion | Python/SQL | Idempotent adapter | 3 |
| 5 | Product | Django/DRF | Users/workspaces | 6 (after M1, ADR-012) |
| 6 | API | FastAPI | M1 vertical slice | 4 |
| 7 | Derivation | Python/SQL | Possessions | 6 |
| 8 | Analytics | Django/FastAPI | M2 MVP | 5, 7 |
| 9 | Recovery | PostgreSQL | Rebuild/corrections | 8 |
| 10 | SQL performance | PostgreSQL | Baselines/indexes | 8 |
| 11 | Async jobs | Celery/RabbitMQ | Backfills | 9 |
| 12 | Caching | Redis | Measured speedup | 10 |
| 13 | Event fan-out | Kafka if justified | M3 independent consumers | 9,11 |
| 14 | Near-live replay | WebSockets | Replay stream | 13 or suitable simpler pipeline |
| 15 | Security | Django/FastAPI | RBAC/API keys | 6 onward; before public deploy |
| 16 | Failure tests | pytest/Testcontainers | Recovery suite | relevant components |
| 17 | Containers | Docker | Reproducible stack | implemented components |
| 18 | CI/telemetry | Actions/OTel | Pipeline/dashboards | 17 (basic CI may start earlier) |
| 19 | Cloud | AWS/Terraform | M4 deployment | 15–18 |
| 20 | Benchmarks | k6/Locust | Measured improvements | 19 or realistic local test env |
| 21 | Portfolio | Docs/GitHub | M5 handoff-ready project | 20 |

**First action:** create repository and commit the charter (`docs/PROJECT_SPEC.md`) specifying MVP and non-goals. Do not install infrastructure first.
