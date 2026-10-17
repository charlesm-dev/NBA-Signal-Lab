# NBA Signal Lab — Architecture

System components, ownership boundaries, data flow, technology admission rules and engineering standards. Scope lives in [PROJECT_SPEC.md](PROJECT_SPEC.md); the reasoning behind each decision lives in [DECISIONS.md](DECISIONS.md) (referenced as ADR-NNN).

**Status:** design only. Nothing below is implemented until the [development log](DEVELOPMENT_LOG.md) says so.

## 1. System overview

```text
NBA.com via nba_api  (private use only — ADR-014)
            |
      Provider adapter            <- only place provider-specific formats/IDs appear
            |
  Validate / normalize / dedupe   <- ingestion package + CLI (batch work, not HTTP)
            |
   PostgreSQL — NBA-owned data
       |                 |
  Derived analytics     FastAPI: NBA data & analytics API (REST; WebSockets later)
  (possessions, etc.)
       |
  Replayable state
       |
  [conditional] Celery/RabbitMQ for long backfills/rebuilds
  [conditional] Kafka for independent durable event consumers
  [conditional] Redis for proven hot reads

Django + DRF control plane
  users / workspaces / membership / saved analyses / admin / audit
            |
   PostgreSQL — Django-owned data
            |
   Auth/token boundary  ->  FastAPI verifies identity and workspace scope

[progressive] Docker Compose -> CI -> observability -> AWS/Terraform
```

## 2. Component boundaries (FIXED — ADR-002, ADR-003)

### Django — control plane

Owns identity and authentication, workspaces and membership, object-level authorization, saved analyses, watchlists, product settings, audit history, Django admin, and future API keys/subscriptions **only if needed**. Django ORM and Django migrations own these tables. DRF is the HTTP interface for product workflows.

Django must **not** perform NBA analytical computation.

### FastAPI — NBA data plane

Owns NBA data access and analytical endpoints, later replay/WebSockets and any inference endpoint. Pydantic models define API contracts.

### Ingestion — NBA data plane, not HTTP (ADR-010)

Provider fetching, validation, normalization and derivation are **batch operations**, implemented as a Python package with a CLI that lives alongside the FastAPI app and shares its data models. They are not FastAPI request handlers. Later, long-running runs may move to background workers (Stage 11) — the package is designed so that move only changes *who calls it*.

### Ownership rules

- Neither framework writes the other's tables. No shared ORM models across the boundary.
- Cross-boundary interaction uses explicit HTTP/internal contracts or carefully defined read-only interfaces.
- Provider-specific code lives only in adapters; provider IDs live only in mapping tables (`ProviderEntityMapping`), never as primary keys of canonical entities.

### Why two frameworks? (ADR-002)

Django is efficient for relational user/business workflows, permissions and admin. FastAPI gives an explicit, independently scalable NBA data/analytics boundary with typed contracts, async-friendly provider I/O and streaming endpoints. The cost — two processes, two dependency sets, a cross-service identity problem — is accepted deliberately and kept small: one monorepo, clear table/migration ownership, explicit contracts, no duplicated auth, co-deployed where practical.

## 3. Data storage

- **One PostgreSQL instance initially**, with **separate schemas** (proposed: `app` for Django, `nba` for NBA data) or equivalently explicit table ownership. Final choice at Stage 3 (ADR-007).
- **Persistence for the NBA schema — PROPOSED:** SQLAlchemy + Alembic. The trade-off (two ORMs and two migration systems against one database vs. one migration system that blurs ownership) is decided at Stage 3 and recorded as ADR-007.
- **Local PostgreSQL** runs in Docker Compose (Postgres-only) from Stage 3 onward; full containerization of the app is Stage 17 (ADR-011).
- **Tests run against real PostgreSQL**, never SQLite substitutes.
- **Raw NBA data stays local** (gitignored `data/`). The public repo holds code and small synthetic fixtures only (ADR-014).

## 4. Data model (PROPOSED — depends on Stage 2 source findings)

| Area | Entities |
|---|---|
| Core NBA | `Team`, `Player`, `Season`, `RosterMembership`, `Game`, `ProviderEntityMapping`, `IngestionBatch`, raw source payload metadata |
| Source statistics | `GameEvent` (provider event ID, game, period, game clock, sequence/order, event type, participants, raw reference), `PlayerGameStats`, `TeamGameStats` |
| Derived | `Possession` (game, team, event span, points, outcome, derivation version); optionally `GameStateSnapshot`, rolling-metric projections and lineup stints **only if source completeness supports them** |
| Django product | `User`, `Workspace`, `WorkspaceMembership`, `SavedAnalysis`, `Watchlist`, `AuditEvent`, data-source configuration if relevant |
| Operational | ingestion/rebuild job status, source revisions, parser/algorithm versions; later outbox and consumer checkpoints |

**Known hard problems:** stable cross-source identity; play-by-play ambiguity; possession/free-throw/offensive-rebound logic; late source corrections; reproducible historical metrics; out-of-order events; concurrent reprocessing; efficient game-state queries; cache invalidation; trustworthy failure recovery.

## 5. Identity across the boundary (AGREED direction, details UNRESOLVED — ADR-005)

Django is the identity authority. FastAPI verifies signed short-lived tokens (or another agreed mechanism) and enforces workspace scopes. Token format, revocation and service-to-service auth are decided at Stage 15. Role claims must not be trusted forever — permission changes and revocation must take effect.

## 6. Technology admission rules (ADR-008)

A technology enters the system only through: **real problem → baseline measurement → appropriate solution → verified improvement**.

| Component | Problem it solves | Admission criterion |
|---|---|---|
| PostgreSQL | canonical relational NBA/event data, constraints, SQL analytics | required from Stage 3 |
| Redis | repeated expensive/hot reads; optional rate limits | baseline proves need; safe on cache miss/outage |
| Celery + RabbitMQ | long-running backfills/rebuilds beyond the HTTP lifecycle | measured synchronous pain; clear task semantics |
| Kafka | independent durable event fan-out/replay/projections | multiple real consumers with ordered/replayable needs; not MVP |
| FastAPI WebSockets | live-style updates | replay product requirement plus reconnect design |
| S3 | durable raw payload archive | rights, size and recovery justify object storage |
| External search engine | search beyond PostgreSQL | measured PostgreSQL limitation; otherwise omit |
| Kubernetes | orchestration at scale | no default justification; prefer Compose/ECS |
| AWS ECS/Fargate, RDS, etc. | realistic cloud operation | after reproducible local services and budget review |

If a conditional component is never justified, record **why** in DECISIONS.md — that is a valid engineering outcome.

## 7. Engineering standards

- **Correctness:** database constraints; provider-independent IDs; idempotent ingestion; parser/algorithm versioning; deterministic replay; no silent event loss; basketball metric reconciliation against golden fixtures.
- **Testing throughout:** parser/unit tests; PostgreSQL migration and constraint tests; FastAPI/DRF contract tests; possession edge cases; worker/broker integration; failure, concurrency and security tests; load tests later. Coverage percentage alone is not evidence.
- **Security:** object-level workspace authorization; secure session/token handling; hashed, scoped API keys; TLS; CORS/CSP where relevant; secrets outside Git; dependency scanning; least-privilege IAM; rate limiting; no secrets in logs; respect data licenses and provider quotas. Passwords use a slow password hash (Django's default PBKDF2, or Argon2/bcrypt) — never MD5/SHA.
- **Reliability:** bounded retries with backoff and jitter; retryable vs permanent errors; idempotent workers; duplicate/out-of-order handling; correction/replay strategy; poison-event quarantine; defined DB-failure behavior; explicit RPO/RTO assumptions for staging.
- **Observability:** correlation IDs; structured logs; p50/p95/p99 API and DB latency; error rate; ingestion volume; parser failures; queue wait; worker throughput; consumer lag; cache hit ratio; event-to-WebSocket delay; tracing where async hops warrant it.
- **Performance:** baseline before optimization; stable fixtures and environment; query counts and plans; p95/p99 and error rate at each load point; record actual results only.
- **Delivery:** meaningful commits, issues, branches, PR self-review, CI checks, milestone tags, up-to-date README/API docs/ERD/ADRs, reproducible setup.
- **Scaling:** reason about growth hypothetically; never claim real large-scale usage. Optimize queries and the data model before caching; add queues/streams/orchestration only when the product needs them.

## 8. Repository layout

**UNRESOLVED** until the first code lands (Stage 2 spike script; Stage 3 schema). A monorepo is AGREED (ADR-004). The layout will be proposed and recorded here before it is created.
