# Architecture Decision Records

Significant decisions and their rationale, newest decisions appended at the end. Each record is short: context, decision, alternatives, consequences. A decision is changed by adding a new ADR that **supersedes** the old one — old records are not rewritten.

**Statuses:** FIXED · AGREED · PROPOSED · CONDITIONAL · SUPERSEDED. ADR-001 through ADR-009 were carried over from the original planning handoff (pre-2026-10-16); later records were made during development.

| ADR | Decision | Status |
|---|---|---|
| 001 | NBA derived analytics, not a scores wrapper | FIXED |
| 002 | Python + Django + FastAPI | FIXED |
| 003 | Django control plane / FastAPI NBA data plane | FIXED |
| 004 | Monorepo; no premature microservices | AGREED |
| 005 | Django is identity authority; FastAPI verifies tokens | AGREED (scheme unresolved) |
| 006 | PostgreSQL is the canonical store | FIXED |
| 007 | SQLAlchemy + Alembic for the NBA schema | PROPOSED — decide at Stage 3 |
| 008 | Infrastructure admitted only on measured need | AGREED |
| 009 | Data feasibility gate; historical replay instead of live feed | FIXED |
| 010 | Ingestion is a package + CLI, not HTTP endpoints | AGREED |
| 011 | Local Postgres via Compose from Stage 3; basic CI early | AGREED |
| 012 | Django control plane does not gate M1 | AGREED |
| 013 | Documentation structure | AGREED |

---

## ADR-001 — NBA derived analytics, not a scores wrapper

- **Status:** FIXED (handoff)
- **Context:** Generic CRUD or API-wrapper projects don't demonstrate deep backend engineering.
- **Decision:** The product's value is the pipeline from source data to deterministic derived basketball state and historical analysis. Possession reconstruction plus historical game-state comparison is the **recommended** first differentiating analytic; the exact metric set is unresolved.
- **Alternatives:** scores/standings app; fantasy/betting tool — rejected as shallow or out of scope.
- **Consequences:** Hard, testable domain problems (event state machines, corrections, replay). Correctness work dominates the MVP.

## ADR-002 — Python + Django + FastAPI

- **Status:** FIXED (handoff)
- **Context:** The developer wants Python backend depth and a deliberate two-plane design.
- **Decision:** Use Django (+ DRF) and FastAPI (+ Pydantic), both in Python.
- **Alternatives:** Django only (simpler, but no separate typed data/analytics boundary); FastAPI only (no admin/auth/product scaffolding); other ecosystems (Java/Spring, Go, Node) — valid but not chosen.
- **Consequences:** Two processes and dependency sets; a cross-service identity problem to solve; must keep both simple and co-deployable. The two-framework cost is accepted deliberately and must be explainable in interviews.

## ADR-003 — Django control plane / FastAPI NBA data plane

- **Status:** FIXED (handoff)
- **Decision:** Django owns users, workspaces, permissions, saved analyses, admin, audit. FastAPI owns NBA data access and analytics APIs (and later replay/WebSockets). Neither writes the other's tables; cross-boundary access via explicit contracts.
- **Consequences:** Clear table and migration ownership; no shared ORM models; analytical computation never lives in Django.

## ADR-004 — Monorepo; no premature microservices

- **Status:** AGREED (handoff)
- **Decision:** One repository containing both applications, shared tooling and docs.
- **Alternatives:** separate repos per service — more overhead, harder integration, no benefit at this scale.
- **Consequences:** Simple integration and CI; boundaries are enforced by code structure and review, not repo walls.

## ADR-005 — Django is identity authority; FastAPI verifies tokens

- **Status:** AGREED direction; token scheme, revocation and service-to-service auth UNRESOLVED (decide at Stage 15, with a minimal version possibly earlier)
- **Decision:** Users authenticate with Django. FastAPI verifies signed short-lived tokens (or an agreed equivalent) and enforces workspace scope.
- **Consequences:** No duplicated user store. Must handle revocation and permission changes — role claims cannot be trusted forever.

## ADR-006 — PostgreSQL is the canonical store

- **Status:** FIXED direction (handoff)
- **Decision:** PostgreSQL for canonical NBA data, derived state and product data.
- **Alternatives:** NoSQL-first — rejected; the domain is relational (identities, constraints, transactions, SQL analytics such as window functions).

## ADR-007 — SQLAlchemy + Alembic for the NBA schema

- **Status:** PROPOSED — decide at Stage 3
- **Context:** Django ORM + migrations will own product tables. The NBA schema is read and written by FastAPI and the ingestion package.
- **Options to evaluate at Stage 3:**
  1. SQLAlchemy + Alembic own a separate `nba` schema (clear ownership; two ORMs/migration systems against one DB).
  2. Django migrations own all tables; FastAPI reads NBA tables via SQLAlchemy/SQL (one migration system; blurs ownership).
  3. Another persistence approach with explicit justification.
- **Evidence note:** the one source-verified posting in the job research (Tessera Labs, verified in a prior audit) named FastAPI + SQLAlchemy — a single data point, not a reason by itself.

## ADR-008 — Infrastructure admitted only on measured need

- **Status:** AGREED (handoff)
- **Decision:** Every technology beyond the fixed stack follows: real problem → baseline → solution → verified improvement. CONDITIONAL components: Celery/RabbitMQ (long backfills), Redis (measured hot reads), Kafka (multiple independent durable consumers — RabbitMQ alone may suffice), WebSockets (replay product need; polling is the alternative), AWS ECS/RDS/S3 + Terraform (after local reproducibility and budget review; Kubernetes not justified by default), predictions/ML (optional, late, never at the expense of correctness).
- **Consequences:** Some components may never be built; when that happens, record why. Keyword coverage is never a justification.

## ADR-009 — Data feasibility gate; historical replay instead of live feed

- **Status:** FIXED (handoff)
- **Decision:** Before final schema design, verify a legally appropriate source and obtain one real complete game. Do not claim licensed live access; demonstrate live-style behavior by replaying historical games.
- **Consequences:** Stage 2 blocks Stage 3. Scope shrinks if play-by-play detail (e.g. substitutions, ordering) is unavailable. Public-demo intent (developer, 2026-10-16) means redistribution/display rights must also be checked.

## ADR-010 — Ingestion is a package + CLI, not HTTP endpoints

- **Status:** AGREED (2026-10-16)
- **Context:** The handoff assigned "ingestion" to FastAPI. Imports and backfills are batch work that can run for minutes and must be retryable; tying them to an HTTP request lifecycle is fragile.
- **Decision:** Ingestion/normalization/derivation live in a Python package on the NBA data-plane side, invoked by a CLI, sharing models with the FastAPI app. FastAPI serves data; it does not run imports inside requests.
- **Alternatives:** HTTP-triggered synchronous imports — rejected (timeouts, no retry semantics).
- **Consequences:** Moving to background workers later (Stage 11) changes only the caller, not the ingestion code. This clarifies ADR-003; it does not change the boundary.

## ADR-011 — Local Postgres via Compose from Stage 3; basic CI early

- **Status:** AGREED (2026-10-16)
- **Context:** The roadmap needs PostgreSQL at Stage 3 but introduces containers at Stage 17 and CI at Stage 18.
- **Decision:** Add a Postgres-only Docker Compose service at Stage 3, and basic CI (lint + tests) once there is code to check. Stages 17–18 mature these into full containerization and CI/observability.
- **Consequences:** Reproducible database from day one; quality checks don't wait until late stages.

## ADR-012 — Django control plane does not gate M1

- **Status:** AGREED (2026-10-16)
- **Context:** The roadmap master table made Stage 6 (FastAPI, M1) depend on Stage 5 (Django), but M1's definition contains no Django work.
- **Decision:** Order becomes 1 → 2 → 3 → 4 → 6 (M1) → 5 → 7 → 8. Django is built after M1 and before Stage 8, the first stage that requires saved analyses.
- **Consequences:** A working real-data vertical slice arrives sooner. Stage numbers are kept unchanged to avoid breaking references.

## ADR-013 — Documentation structure

- **Status:** AGREED (2026-10-16)
- **Decision:** `docs/PROJECT_SPEC.md` serves as the Stage 1 project charter (replacing the planned `docs/project-charter.md`). This file replaces the planned `docs/adr/` directory with numbered ADRs in one file. `docs/DATA_SOURCES.md` is created in Stage 2 (replacing the planned `docs/data-source-feasibility.md`). The original handoff files are preserved in `docs/archive/` until their content is confirmed incorporated.
- **Consequences:** Fewer, non-overlapping documents; `CLAUDE.md` points to them rather than duplicating them.
