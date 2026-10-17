# NBA SIGNAL LAB — Project Handoff for Claude Code

**Prepared:** 2026-10-16. **Status:** user has chosen the project and Claude Code as primary development assistant; **no implementation has been confirmed**. This is a knowledge-transfer document, not authorization to begin writing code.

**Companion documents:** [`DEVELOPMENT_ROADMAP.md`](DEVELOPMENT_ROADMAP.md) (21 ordered stages with gates, tests and decisions) and [`JOB_MARKET_REQUIREMENTS.md`](JOB_MARKET_REQUIREMENTS.md) (recoverable 24/48 sample competency frequencies, original posting references, caveats). Read both before suggesting architecture changes.

## 1. Executive brief and vision

**NBA Signal Lab** is an NBA-specific, backend-first **basketball event-data ingestion, normalization, historical analysis and game-state intelligence platform**. It should consume usable NBA schedules, team/player/game/box-score/play-by-play information, preserve canonical identities and provenance, reconstruct basketball state (especially possessions), derive statistics and historical comparisons not directly exposed by the provider, and serve analytical APIs. Later it can replay historical games as near-live streams, drive alerts and optionally power model inference/predictions **only if data and validation justify it**.

The project is **not** a basic scores app, standings dashboard, fantasy clone, betting app, a UI wrapped around an NBA API, or a generic CRUD project with basketball branding. Its distinguishing value is the engineered pipeline: **source data → normalized event history → deterministic derived basketball state → queryable historical intelligence → optionally near-live/replay experiences**. The NBA domain is indispensable, not cosmetic.

**Intended users (not yet validated or exhaustively specified):** NBA enthusiasts and analysts who want richer historical/game-state analysis; potentially developers consuming its APIs. Exact target persona, UX, and monetization are **unresolved**. Primary objective is a substantial, credible, portfolio-quality backend project, **not** a commercial launch or claimed professional-equivalent experience.

### Product and scope status labels

- **FIXED:** Python + Django + FastAPI; NBA is core; backend depth, correctness, incremental architecture, serious tests, real performance evidence; completed Roadmap.sh topics interpreted correctly; start with data feasibility; no invented access to proprietary feeds.
- **AGREED roadmap target:** canonical NBA entities, play-by-play normalization, possession reconstruction, historical game-state queries, player/team rolling statistics, Django users/workspaces/saved analyses; provider adapters and idempotent imports; later optional replay/streaming.
- **PROPOSED implementation choices:** Django REST Framework, SQLAlchemy/Alembic for analytics-owned schema, Celery/RabbitMQ, Redis, Kafka, AWS ECS/RDS/S3, Terraform, OTel/Grafana, pytest/Testcontainers, k6/Locust. These are not installed or irreversible decisions.
- **UNRESOLVED:** specific provider/license, exact analytics definitions, data volume, precise source update schedule, user experience, auth exchange, persistence/migration ownership, budget, live-data rights and whether advanced infrastructure is actually needed.

## 2. Functional requirements and workflows

### MVP — required after data feasibility

1. **Acquire a usable NBA historical dataset.** Evaluate one or more legally appropriate sources; obtain at least one real complete game with play-by-play if supported, then a meaningful historical slice (initially one season recommended). Capture provenance, external IDs, ingestion timestamps and parser versions.
2. **Canonical identity and domain.** Represent seasons, teams, players, roster membership over time, games, box-score stats and ordered play-by-play; map provider IDs to stable internal IDs.
3. **Repeatable ingestion.** Import teams → players → schedule/games → box scores → events, validate, upsert/version appropriately, prevent duplicates and surface failed/partial batches.
4. **Game/event access.** FastAPI endpoints for team, player, game, game events and player game history; pagination, versioned responses, errors and API documentation.
5. **Derived NBA intelligence.** Normalize event taxonomy; deterministically reconstruct possessions with explicit handling of rebounds, fouls, free throws, substitutions, period changes and data errors; reconcile scores and validate against known games.
6. **Historical analytical queries.** Historical game-state similarity (e.g. fourth quarter, trailing by ten, five minutes left), rolling player/team form, scoring runs/comeback situations, game comparisons. Exact formulas and filters require domain decisions and validation.
7. **Product workflows.** Django login, workspaces, permissions, saved analysis configurations, watchlists as useful, admin for managing product data. Users can retrieve and save analytical results/configurations.
8. **Reproducibility.** Source fixtures, migrations, repeatable rebuild of derived state, automated tests, baseline latency/query metrics and developer documentation.

### Advanced — conditional, introduced after measurable need

- Backfill and recomputation jobs outside HTTP requests; durable retries and visible status (Celery/RabbitMQ candidate).
- Redis cache for measured expensive/hot analytics; explicit TTL/invalidation and no correctness dependence on cache.
- Independent event consumers for projections/alerts/model inference **only if** fan-out/replay justifies Kafka; ordered by game, idempotent consumers, schema/version and publish-reliability design.
- Historical event replay at real/accelerated clock speed, WebSockets for game-state updates, reconnect snapshots and lag measurement; **not** a claim of licensed live NBA access.
- Alerts/watchlists and optional model inference/prediction; model training and predictive accuracy claims are not MVP requirements.
- Scaled historical backfills, correction propagation, object storage of permitted raw payloads, observability, cloud staging, reproducible deployment, load/failure testing.

### Example end-to-end workflows

**Analyst:** sign in via Django → select historical game state → FastAPI queries canonical/derived NBA data → see comparable game situations → save query/workspace in Django.

**Data maintainer:** fetch permitted source game → record ingestion batch/raw metadata → validate and normalize → write canonical entities/events → reconstruct possessions → expose versioned API; reprocess on source correction.

**Advanced replay subscriber:** select archived game → replay ordered events → consumers update projections → WebSocket delivers state changes → reconnecting client requests authoritative snapshot.

## 3. Architecture and boundaries

```text
Historical/open/provider NBA source(s)
            |
      Provider adapter
            |
  Validate / normalize / dedupe
            |
    PostgreSQL (NBA-owned data)
       |               |
  Derived analytics      FastAPI NBA data/analytics API
       |               |
  Replayable state    REST; optional WebSockets
       |
 [later] Celery/RabbitMQ for requested backfills
 [later] Kafka for independent durable event consumers
 [later] Redis for proven hot reads

Django + DRF control plane
  Users / workspaces / membership / saved analyses / admin / audit
            |
    PostgreSQL (Django-owned tables)
            |
      Auth/token boundary to FastAPI

[later] Docker / CI / OpenTelemetry / AWS / Terraform
```

### Django responsibilities — fixed boundary

Own identity, authentication, workspace membership, object-level authorization, saved analyses, watchlists, product settings, audit history, admin and future API key/subscription controls **only if needed**. Django ORM/migrations own its tables. DRF is a recommended HTTP interface for product workflows.

### FastAPI responsibilities — fixed boundary

Own NBA provider/data ingestion API boundary, NBA data access and analytical endpoints, potentially high-volume/async I/O, replay/WebSockets and inference endpoint later. Pydantic defines API contracts. SQLAlchemy/Alembic is a **proposal**, not a mandate, for FastAPI-owned data models/migrations.

### Why both?

Django is efficient for complex relational user/business workflows and admin; FastAPI gives a clear independently scalable NBA ingestion/analytical API boundary, async-friendly provider I/O and streaming endpoints. **Using two frameworks has operational cost**; maintain one monorepo, clear table/migration ownership, explicit contracts, and avoid duplicating auth or creating many services. If real workload never warrants separate scaling, two processes remain a requirement of this project but should be kept simple and co-deployed where practical. No cross-framework direct writes to each other's owned tables.

### Communication and identity

At first, services may share one PostgreSQL instance with **separate schemas or explicitly segregated table ownership**. FastAPI reads NBA data; Django owns user/workspace records. Cross-boundary interaction should use explicit HTTP/internal contracts or carefully defined read-only interfaces. Django is the eventual identity authority; FastAPI verifies signed tokens or another agreed mechanism and enforces workspace scopes. Exact token, revocation and service-to-service design are unresolved. Do not encode roles once and forget revocation/permission changes.

### Infrastructure choices and admission tests

| Component | Problem it solves | Admission criterion |
|---|---|---|
| PostgreSQL | canonical relational NBA/event data, constraints, SQL analytics | required from early stages |
| Redis | repeated expensive/hot reads, optional rate limits | baseline proves need; safe cache miss/outage |
| Celery + RabbitMQ | long-running backfills/rebuilds beyond HTTP lifecycle | measured sync pain and clear task semantics |
| Kafka | independent durable event fan-out/replay/projections | multiple real consumers, ordered/replayable event need; not MVP |
| FastAPI WebSockets | live-style updates | replay/current-state product requirement and reconnect design |
| S3 | durable raw payload/archive | rights, size and recovery justify object storage |
| External search engine | complex search beyond PostgreSQL | actual query/search limitation measured; otherwise omit |
| Kubernetes | orchestration at scale | no default justification; prefer ECS/Compose |
| AWS ECS/Fargate, RDS, etc. | realistic cloud operation | after reproducible local services and budget review |

## 4. NBA data: confirmed limitations and feasibility gate

**No specific NBA API, paid feed, dataset, license, provider contract, rate limit, historical depth or live-data entitlement has been verified in the accessible record.** The earlier roadmap **proposed** a feasibility spike; it did not report its completion. Claude must not silently choose an endpoint or assert access. Sources to investigate include official/public interfaces, third-party NBA data vendors, openly licensed historical datasets and permitted archival files. Web accessibility alone does **not** grant redistribution/commercial rights; scraping must respect terms and technical limits.

**Required source evaluation fields:** source/provider, official status, pricing, legal terms, personal/noncommercial display/storage/redistribution rules, stable player/team/game IDs, schedule/box-score/play-by-play coverage, substitution events, ordering/timestamps, historical seasons, missingness, late corrections, API quotas, update latency, permitted caching, source attribution and fallbacks.

**Frequency policy:** historical backfill on demand; completed-game corrections via bounded periodic reconciliation as permitted; schedule/roster refresh based on provider's documented update cadence; live/near-live only if licensed/feasible. No numeric polling frequency is agreed. Replayed historical games are the legitimate fallback for testing real-time engineering.

### Data model (proposed, source-dependent)

- **Core:** `Team`, `Player`, `Season`, `RosterMembership`, `Game`, `ProviderEntityMapping`, `IngestionBatch`, raw source payload metadata.
- **Source statistics:** `GameEvent` (provider event ID, game ID, period, game clock, sequence/order, event type, participating entities, raw reference), `PlayerGameStats`, `TeamGameStats`.
- **Derived:** `Possession` (game, team, event span, points, outcome, derivation version); optionally `GameStateSnapshot`, rolling metric projections and lineup stints **only if source completeness supports them**.
- **Django product:** `User`, `Workspace`, `WorkspaceMembership`, `SavedAnalysis`, `Watchlist`, `AuditEvent`, data-source configuration if relevant.
- **Operational:** ingestion/rebuild job status, source revisions, algorithm version, optional outbox and consumer checkpoints later.

**Hard problems:** stable cross-source identity, NBA play-by-play ambiguity, possession/FT/offensive rebound logic, late source corrections, reproducible historical metrics, out-of-order events, concurrent reprocessing, efficient time-series/game-state queries, cache invalidation and trustworthy failure recovery.

## 5. Development roadmap

The complete stage-by-stage guide, with objective, build tasks, rationale, technologies, reinforcing vs learning, execution order, decisions, definition of done, tests, pitfalls and interview knowledge, is in **`DEVELOPMENT_ROADMAP.md`**. Preserve the following order:

| # | Stage | Gate / result |
|---|---|---|
| 1 | Charter, repo, GitHub workflow | MVP/non-goals documented |
| 2 | NBA source feasibility | real complete game fixture, legal/technical assessment |
| 3 | Canonical schema | ERD, migrations, uniqueness/FKs |
| 4 | Synchronous adapter/import | repeat import without duplication |
| 5 | Django control plane | users, workspaces, saved analyses, permissions |
| 6 | FastAPI data API | **M1** real game queryable, tests, baseline |
| 7 | Play-by-play/possessions | validated state machine and golden games |
| 8 | Historical analytics | **M2** usable MVP, baseline metrics |
| 9 | Corrections and replay | rebuildable derived state |
| 10 | SQL profiling | measured index/query improvements |
| 11 | Celery jobs | long backfills decoupled and recoverable |
| 12 | Redis | proven read speedup, safe invalidation |
| 13 | Kafka only if warranted | **M3** reliable independent consumers |
| 14 | Replay/WebSockets if warranted | deterministic near-live demo |
| 15 | Cross-service auth/security | scoped RBAC and attack tests |
| 16 | Failure testing | recovery matrix verified |
| 17 | Docker/Compose | fresh-clone reproducibility |
| 18 | CI/observability | validated PRs, traces and metrics |
| 19 | AWS/Terraform deployment | **M4** secure reproducible staging |
| 20 | Load/scale experiments | real before/after improvements |
| 21 | Documentation/portfolio | **M5** complete engineering story |

**Scheduling nuance:** basic tests, auth, Docker and CI can be introduced incrementally earlier than their dedicated hardening stages; dedicated stages mean maturation, not postponing all quality/security until late. Kafka is conditional; if unjustified, record why and use a simpler event pipeline. Do not block project completion on unjustified tools.

## 6. User's Roadmap.sh progress — authoritative interpretation

**Rule: greyed-out and crossed-out means COMPLETED; purple-filled means CURRENTLY LEARNING; non-grey/non-crossed without purple means not completed/not currently learning.** Do not mark completed topics as missing. User explicitly corrected earlier misclassification.

### Completed — reinforce through project

- **API/web:** REST, JSON APIs.
- **Authentication:** authentication fundamentals, JWT, OAuth, Basic Authentication, Token Authentication, Cookie-Based Authentication.
- **Hashing/security:** MD5, SHA, scrypt, **bcrypt**; HTTPS, OWASP Risks, CORS, SSL/TLS, CSP, Server Security; general web/server security concepts previously listed as completed where crossed out. **Important:** MD5/SHA are knowledge, not recommendations for password storage; use secure password hashing.
- **Servers/cache:** learn about Web Servers, Nginx, Caching, HTTP Caching.
- **AI fundamentals:** basics, how LLMs work, AI vs traditional coding, embeddings, vectors, RAGs. These are optional domain tools, not a reason to bolt LLMs onto the MVP.

### Currently learning (purple)

**PostgreSQL, Redis, Claude Code.** PostgreSQL is essential early; Redis is intentionally delayed until justified; Claude Code is a development assistant, not an application runtime dependency.

### Not completed / not currently learning (per screenshot, not an obligation)

MariaDB, MS SQL, SQLite, Oracle, MySQL as a separate database, database migrations, N+1 problem; SOAP, gRPC, GraphQL, OpenAPI Specs and some broader API roadmap topics; OpenID, SAML; Apache, Caddy, MS IIS; Memcached; Cursor, Copilot, Antigravity. Some items might be revisited for job-market value, but **user does not want the project optimized to complete every unchecked roadmap box**. OpenAPI is useful as a by-product of FastAPI; alternatives like SOAP/SAML/IIS should not be added for checkbox coverage.

**Ambiguity:** Earlier screenshot interpretation and user correction differ on some greyed/crossed details; the user's explicit completed list above takes precedence. If another item matters, ask the user to confirm instead of asserting.

## 7. Learning objectives and job-market relevance

**Reinforce:** REST/JSON, JWT/OAuth/authn, HTTPS/TLS, CORS/CSP/OWASP, hashing awareness, Nginx/web server fundamentals, HTTP caching, basic AI/embeddings knowledge only if relevant.

**Learn/practice deeply:** Python backend production patterns; Django/DRF; FastAPI/Pydantic; PostgreSQL/SQL, schema design, migrations, N+1, EXPLAIN/indexing/window functions, transactions/isolation; provider adapters and idempotency; play-by-play state machines; reliable background jobs; Redis consistency; event-driven design, consumer semantics and distributed recovery if needed; Docker, AWS/IAM, Terraform, CI/CD, structured logs, metrics/traces, test automation, security/RBAC, performance/load testing, technical writing and ADRs.

**Research summary:** prior 48-posting aggregate (2025–2026 early-career backend-oriented roles) reported scale/reliability **44/48**, APIs **37/48**, cloud **32/48**, distributed systems **30/48**, relational SQL **29/48**, testing **26/48**, Python **25/48**, containers **22/48**, messaging **17/48**, caching **10/48**, CI/CD **10/48**. Other exact combined counts are not recoverable. See `JOB_MARKET_REQUIREMENTS.md` for original sample comparison, links and evidence caveats. No project can satisfy Java/Spring/Go/.NET ecosystem-specific experience literally or substitute for production employment.

## 8. Engineering standards

- **Correctness:** database constraints; provider-independent IDs; idempotent ingestion; parser versioning; deterministic replay; no silent event loss; basketball metric reconciliation and golden fixtures.
- **Testing throughout:** parser/unit tests; PostgreSQL migration/constraint tests; FastAPI/DRF contract/API tests; possession edge cases; worker/broker integration; failure, concurrency and security tests; load tests later. Do not rely on coverage percentage alone.
- **Security:** object-level workspace authorization, secure session/token handling, API-key hashing/scopes, TLS, CORS/CSP where relevant, secrets outside Git, dependency scanning, least-privilege cloud IAM, rate limiting, no secrets in logs; respect data license and provider quotas.
- **Reliability:** bounded retries with backoff/jitter, retryable vs permanent errors, idempotent workers, duplicate/out-of-order handling, correction/replay strategy, poison-event quarantine, DB failure behavior, explicit RPO/RTO assumptions for demo/staging.
- **Observability:** correlation IDs, structured logs, p50/p95/p99 API and DB latency, error rate, ingestion volume, parser failures, queue wait, worker throughput, consumer lag, cache hit, event-to-WebSocket delay; distributed tracing where asynchronous hops warrant.
- **Performance:** record baseline before optimization, stable test fixtures and environment, query counts/plans, p95/p99 and error rate at each load point; document changes and actual results only.
- **Delivery:** meaningful Git commits, issues, branches, PR self-review, CI checks, milestone tags, up-to-date README/API docs/ERD/ADRs, clean setup, reproducible containers, staged AWS deployment, backup/rollback rehearsal.
- **Scaling:** reason hypothetically from small usage to high usage, but do not claim actual million-user scale. Optimize queries and data model before caching; add queues/streams/orchestration only when product needs them.

## 9. Decisions log and open questions

| Decision | Status | Reasoning / alternatives |
|---|---|---|
| NBA-focused derived analytics, not scores wrapper | FIXED | distinctive NBA domain and deep event/data engineering |
| Python + Django + FastAPI | FIXED | exact requested stack; control vs analytical data planes; operational cost accepted |
| PostgreSQL canonical store | FIXED direction | relational identities, transactions, advanced SQL; not NoSQL-first |
| Monorepo, avoid premature microservices | AGREED | simpler integration, independent boundaries still possible |
| Source feasibility before final schema | FIXED gate | avoid building on inaccessible data or unsupported event detail |
| Historical replay as alternative to live feed | AGREED option | avoids claiming proprietary real-time access |
| Possession reconstruction + historical game-state analysis as MVP focus | RECOMMENDED/roadmap target | differentiating, testable NBA analysis; exact metric set unresolved |
| Django ORM for product / SQLAlchemy for NBA schema | PROPOSED | clear ownership; could use different persistence strategy with justification |
| Django identity authority, FastAPI verifies tokens | AGREED direction | avoid duplicated identity; token scheme unresolved |
| Celery/RabbitMQ | CONDITIONAL | background backfills if HTTP lifecycle too slow; simpler alternatives possible |
| Redis | CONDITIONAL | cache only after measured SQL/read bottleneck |
| Kafka | CONDITIONAL | independent durable fan-out and replay; RabbitMQ alone may suffice |
| WebSockets | CONDITIONAL | replay/live-style client experience; polling possible alternative |
| AWS ECS/RDS/S3/Terraform | PROPOSED later | production-like and avoids K8s overhead; costs/budget unresolved |
| Predictions/ML | OPTIONAL later | do not sacrifice analytics correctness or introduce unverifiable claims |

### Critical missing information Claude must obtain/validate

1. **Actual source and license:** no chosen provider, credentials, verified rate limits, season depth, complete play-by-play or live entitlement.
2. **Exact MVP contract:** which possession rules, historical game-state filters, rolling stats and UX are mandatory; original NBA project pitch was not preserved verbatim in the accessible record, so do not invent hidden features.
3. **User's current repository:** no GitHub URL, directory or codebase confirmed. Ask whether one exists before making edits.
4. **Python/SQL/Django/FastAPI proficiency:** Roadmap shows PostgreSQL/Redis in progress, but precise Python experience and framework proficiency not confirmed.
5. **Runtime choices:** Python version, dependency manager, ORM/schema boundaries, dev OS, deployment budget, chosen data source and redistribution policy.
6. **Auditable market research:** full second set of 24 job postings and exact per-posting coded matrix are missing from the accessible conversation record. Prior combined counts can be cited as *reported earlier*, not independently reverified.
7. **Product audience:** analyst vs fan vs developer, whether UI is necessary beyond API docs/admin, authentication UX and API access model.
8. **Timeline:** available weekly hours and desired target release date unknown; roadmap stages are dependency gates, not promised calendar deadlines.

## 10. Success criteria and honest claims

**M1:** permitted real game ingestion, canonical PostgreSQL data, tested FastAPI responses and first baseline.

**M2:** validated possession/game-state analysis over meaningful history, Django product workflows, repeatable ingestion/rebuild, documented API and automated tests.

**M3 (where justified):** measured asynchronous processing and caching, independently consumed events/replay when useful, proven duplicate/retry/recovery behavior; don't install infrastructure without purpose.

**M4:** hardened auth/security, reliable failure handling, Docker/CI/telemetry, reproducible secure cloud staging/deployment, backups and rollback.

**M5:** convincing architecture diagrams, ADRs, meaningful Git history, reproducible benchmarks, **actual** before/after performance evidence, explainable domain correctness and thoughtful tradeoffs. Résumé bullets must use measured numbers, never fabricated claims.

**Not demonstrated:** professional production tenure, true NBA-scale traffic, actual large-team/on-call ownership, real customer incidents, enterprise regulatory certification or access to licensed live data. Portfolio-scale simulation should be labeled as such.

## 11. Current status and instructions for Claude Code

**Confirmed:** user chose NBA Signal Lab, Python + Django + FastAPI, backend-first scope, and Claude Code as primary assistant; prior research and roadmap exist as planning documents. **Not confirmed:** repository created, project charter committed, provider selected, dataset obtained, database/schema implemented, APIs running, any infrastructure provisioned, any tests or benchmarks executed. Treat all as **unimplemented until inspected or user-confirmed**.

**Immediate next action (exactly one):** ask the user to **create the GitHub repository and commit `docs/project-charter.md` defining the MVP and explicit non-goals** (or inspect an existing repo first if user says it exists). Do **not** install Django/FastAPI/Kafka/Redis yet. After that, Stage 2 is the NBA source feasibility spike.

### Claude Code operating protocol

1. Read this handoff and both companions. Inspect repository before assuming code exists.
2. Act as a senior backend mentor: propose **one stage at a time**, explain constraints and review criteria, let user implement core code; don't dump full solutions unless requested.
3. Before each stage, state required vs optional work, key design decisions, tests, definition of done, and one immediate next task.
4. Never silently change fixed stack/product; flag only serious architectural issues and discuss alternatives.
5. Require evidence for completion (tests, commands, sample responses, measurements), not verbal assumptions.
6. Track decisions as ADRs and issues; keep docs in sync with implemented reality.
7. Avoid overengineering, unnecessary tool installation, fabricated source access, fabricated job data and invented benchmark results.
8. Keep a clear distinction between **confirmed**, **recommended**, **conditional**, and **unresolved** at every handoff.

**End of handoff.**
