# Learning Progress

Tracks what the developer actually knows, not what happens to exist in the codebase. Code written by Claude, or code the developer used without explaining it, is **not** evidence of learning.

## Status levels

| Level | Meaning | Evidence required |
|---|---|---|
| **Prior** | Known before this project (self-reported or roadmap.sh) | Developer's statement |
| **New** | Not yet encountered in this project | — |
| **Introduced** | Explained; developer has seen it used | The explanation happened |
| **Practiced** | Developer implemented it in this repo, possibly with help | Commit/PR the developer wrote |
| **Can explain** | Developer explained the why, the tradeoffs and failure modes without prompting, or debugged it independently | Review discussion, noted in the ledger |

A concept moves up only with evidence. Claude proposes changes during `/review-my-code` or `/wrap-session`; the developer confirms them.

## 1. Background (self-reported 2026-10-16)

| Area | Level | Notes |
|---|---|---|
| Programming fundamentals | Strong | Frontend internship |
| JavaScript | Strong | Used heavily at internship; Python concepts can be taught by contrast with JS |
| Git | Strong | Used heavily at internship |
| SQL | Basics | Fundamentals known; joins, constraints, indexes, transactions and window functions to be built up |
| Python | Very little | Needs an on-ramp: virtual environments, packaging, typing, idioms, testing with pytest |
| Django | None | |
| FastAPI | None | |

## 2. roadmap.sh status (authoritative interpretation from handoff)

Rule: greyed-out/crossed-out = completed; purple = currently learning; otherwise not started. The developer's explicit list takes precedence over earlier screenshot readings.

- **Completed — reinforce through the project:** REST; JSON APIs; authentication fundamentals; JWT; OAuth; Basic, Token and Cookie-based authentication; MD5, SHA, scrypt, bcrypt (knowledge only: MD5/SHA are not for password storage); HTTPS; OWASP risks; CORS; SSL/TLS; CSP; server security; web servers; Nginx; caching; HTTP caching; AI basics, LLMs, embeddings, vectors, RAG.
- **Currently learning:** PostgreSQL, Redis, Claude Code.
- **Not started (not an obligation):** MariaDB, MS SQL, SQLite, Oracle, MySQL; database migrations; N+1 problem; SOAP, gRPC, GraphQL, OpenAPI specs; OpenID, SAML; Apache, Caddy, MS IIS; Memcached; Cursor, Copilot, Antigravity.

The project is **not** optimized to tick roadmap boxes. OpenAPI comes naturally with FastAPI; SOAP, SAML, IIS and similar are not added for coverage.

## 3. Concept ledger

Concepts the project will teach, grouped by the stage where they first matter. All start at **New** unless listed as Prior above. Update the level, the date and the evidence when they change.

| Concept | Stage | Level | Evidence / notes |
|---|---|---|---|
| Scoping, non-goals, ADRs | 1 | Introduced | Charter and ADRs drafted by Claude 2026-10-16; developer review pending |
| Python environment and tooling (venv, uv, project layout) | 2 | New | |
| Python idioms vs JavaScript (types, dicts/lists, comprehensions, exceptions, modules) | 2 | New | |
| HTTP clients in Python (HTTPX), timeouts, retries | 2 | New | REST itself is Prior |
| Data licensing, provenance, provider rate limits | 2 | New | |
| Fixtures and smoke tests; pytest basics | 2 | New | |
| Relational modeling: keys, normalization, constraints | 3 | New | SQL basics are Prior |
| Database migrations | 3 | New | roadmap.sh: not started |
| SQLAlchemy / Alembic (or chosen persistence) | 3 | New | |
| Docker Compose for local Postgres | 3 | New | |
| Adapter pattern; provider isolation | 4 | New | |
| Idempotency and transactional upserts | 4 | New | |
| Validation of external payloads | 4 | New | |
| FastAPI, Pydantic, dependency injection | 6 | New | |
| API versioning, pagination, error contracts | 6 | New | REST/JSON Prior |
| Sync vs async I/O in Python | 6 | New | Async in JS is Prior; the Python model differs |
| Latency baselines (p50/p95) and query counts | 6 | New | |
| Django ORM, admin, migrations | 5 | New | |
| DRF, object-level permissions | 5 | New | Authn concepts Prior |
| Event state machines; possession logic | 7 | New | |
| Golden-fixture regression testing | 7 | New | |
| Window functions and aggregation | 8 | New | |
| N+1 queries, EXPLAIN, indexing | 10 | New | roadmap.sh N+1: not started |
| Task queues, at-least-once delivery, idempotent workers | 11 | New | |
| Cache-aside, invalidation, staleness | 12 | New | HTTP caching Prior; Redis in progress |
| Event logs vs queues, outbox, consumer offsets | 13 | New | Conditional stage |
| WebSockets, reconnect, backpressure | 14 | New | Conditional stage |
| Cross-service tokens, RBAC, API-key hashing | 15 | New | JWT/OAuth Prior |
| Failure engineering, backoff with jitter | 16 | New | |
| Container images, health checks | 17 | New | |
| CI pipelines, structured logs, metrics, tracing | 18 | New | |
| AWS (IAM, ECS, RDS), Terraform | 19 | New | |
| Load testing, bottleneck analysis | 20 | New | |

## 4. Session notes

Short, dated entries recording what was learned or demonstrated, newest first.

- **2026-10-16:** Project setup (Stage 1). No implementation yet. Baseline recorded from self-report.
