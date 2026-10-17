# NBA Signal Lab — Backend Job-Market Requirements

Which backend competencies NBA Signal Lab is meant to demonstrate, and how strong the evidence behind that list is. Used to guide technical priorities — **never** to justify adding a tool for keyword coverage (see ADR-008 in [DECISIONS.md](DECISIONS.md)).

## 1. Evidence status — read this first

Prior research (ChatGPT-era, 2025–2026 postings) described **48 early-career backend-oriented postings**: an original 24 and a second 24. The evidence quality differs sharply:

| Evidence item | Status |
|---|---|
| Original 24 — identities | Recovered (company + role) |
| Original 24 — source URLs | 3 of 24 preserved; **not re-verified** |
| Second 24 — identities and requirement summaries | Recovered from earlier conversation context; **unverified** (23 of 24) |
| Second 24 — source URLs | **1 of 24** (Tessera Labs) source-verified by the prior audit; see §4 for Claude's 2026-10-16 recheck |
| Per-competency counts (both batches) | **Historical results of earlier coding**, not re-counted or audited |
| Row-level coding ledger, frozen rubric, dated posting copies | **Missing** |

**Interpretation rules**

- Counts are explicit-mention rates in a **nonrandom convenience sample**, not market prevalence. They support *directional* conclusions only.
- Do not invent missing counts, URLs, quotations or posting identities.
- Do not cite an unverified posting's requirements as fact; cite them as "previously reported".
- Exact Django/FastAPI/Java-Spring frequencies are **not recoverable**.

The full second-batch audit is preserved at [archive/SECOND_BATCH_BACKEND_JOB_RESEARCH.md](archive/SECOND_BATCH_BACKEND_JOB_RESEARCH.md).

## 2. Normalization (coding rubric used historically)

- Backend services/APIs: REST, GraphQL, gRPC and endpoint development (protocol-specific mentions kept separately).
- SQL/relational: PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, schema/ORM.
- Cloud: AWS, Azure, GCP. Containers: Docker/Kubernetes (orchestration distinct where relevant).
- Infrastructure/deployment: IaC, Terraform, ECS/Kubernetes, deployment tooling; CI/CD separately.
- Async/event-driven: Kafka, RabbitMQ, SQS, pub/sub, background processing; task queues vs durable streams distinguished.
- Caching: Redis, Memcached, cache design. Testing: unit/integration/automated. Security: authn, authz, privacy, API security.
- Distributed systems: explicit service coordination. Scale/reliability/performance: explicitly stated expectations.
- Each posting counted at most once per competency; competencies overlap; language mentions are nonexclusive.

**Caveat:** some second-batch categories (security, IaC) were coded with **different definitions** than the first batch. Those are reported separately in §3.2 and are not comparable.

## 3. Competency matrix (historical counts — unaudited)

### 3.1 Comparable categories

Second-batch counts are derived arithmetically (combined − original). "Δ" compares second batch to first batch.

| Competency | Original 24 | Second 24 | Combined 48 | Δ (2nd − 1st) | Project relevance |
|---|---:|---:|---:|---:|---|
| Scalability/reliability/performance | 21 (87.5%) | 23 (95.8%) | **44 (91.7%)** | +8.3 pp | Essential: load tests, replay recovery, latency SLOs |
| Backend APIs/services | 21 (87.5%) | 16 (66.7%) | **37 (77.1%)** | −20.8 pp | Essential: FastAPI/DRF, contracts, pagination |
| Cloud | 16 (66.7%) | 16 (66.7%) | **32 (66.7%)** | 0 pp | Essential/high-value: AWS deployment, IAM |
| Distributed systems | 14 (58.3%) | 16 (66.7%) | **30 (62.5%)** | +8.3 pp | High-value: worker recovery, messaging, projections |
| SQL/relational | 16 (66.7%) | 13 (54.2%) | **29 (60.4%)** | −12.5 pp | Essential: PostgreSQL schema, SQL tuning |
| Automated testing | 12 (50.0%) | 14 (58.3%) | **26 (54.2%)** | +8.3 pp | High-value: golden-game, DB, API, failure tests |
| Python | 12 (50.0%) | 13 (54.2%) | **25 (52.1%)** | +4.2 pp | Strong exact language overlap |
| Containers | 16 (66.7%) | 6 (25.0%) | **22 (45.8%)** | −41.7 pp | High-value: Docker/Compose; no automatic K8s |
| Messaging/event-driven | 8 (33.3%) | 9 (37.5%) | **17 (35.4%)** | +4.2 pp | Useful: Celery/RabbitMQ; Kafka only if justified |
| Caching | 5 (20.8%) | 5 (20.8%) | **10 (20.8%)** | 0 pp | Useful: Redis where measured |
| CI/CD | 5 (20.8%) | 5 (20.8%) | **10 (20.8%)** | 0 pp | Useful, professionally expected: GitHub Actions |

The container drop in batch two does **not** show that Docker lost market value; it may reflect sampling choices.

### 3.2 Non-comparable categories (different definitions per batch)

| Competency | Original 24 | Second 24 (approx., historical) | Note |
|---|---|---|---|
| Security/auth | 5/24 (narrow definition) | ~16/24 (broad definition) | Not a trend — different rubrics. Essential engineering standard regardless. |
| Infrastructure/IaC/deployment | 10/24 (broad) | ~3/24 (narrow IaC) | Not a trend. AWS/Terraform if justified. |
| Data ingestion/pipelines | not preserved | ~12/24 | **Especially strong fit** for NBA Signal Lab |
| Observability | not preserved | ~11/24 | High-value: tracing/metrics/logs |
| AI/LLM | not coded | ~10/24 | Optional; not an MVP requirement |
| Async/background processing, object storage, concurrency/idempotency, gRPC/OpenAPI, search engines | not separately coded | not recoverable | Recurring themes only |

## 4. Source-verified posting

**#35 Tessera Labs — Software Engineering Intern, Backend** — `https://jobs.ashbyhq.com/tessera-labs/dce8d8e5-f169-4bdb-b1c2-f4754fca31e7`

- **Prior audit (2026-10-16, ChatGPT):** reported retrieving the employer-hosted posting. Paraphrased requirements: Python; FastAPI and SQLAlchemy; REST APIs; schema design and SQL; caching; data pipelines; enterprise/AI integrations; scalable/event-driven architecture; AWS or GCP; observability; security. Candidate expectations: Python plus a backend framework (FastAPI, Flask or Django); basic SQL (PostgreSQL or MySQL); interest in distributed systems/cloud; optional queues, caching, AI/ML.
- **Claude recheck (2026-10-16):** the posting page renders client-side and returned no content to a plain fetch. Tessera Labs' public Ashby job-board API listed 24 open roles, including "Software Engineer, Backend" and "Software Engineering Intern, Full Stack", but **not** the posting ID above. **Status: verified by the prior audit only; Claude could not independently confirm it, and it appears to be closed.**

## 5. Second batch — recovery index (UNVERIFIED except #35)

Previously recorded summaries, not posting text. Use as a starting list for a future source-by-source audit, not as evidence.

| # | Company | Role (as previously recorded) | Previously reported technologies |
|---:|---|---|---|
| 25 | Circle | New Grad SWE, Backend | Java/Go, REST, SQL, cloud, distributed services, tests, security, monitoring |
| 26 | DataBahn | New Grad backend/data platform | Ingestion/routing, APIs, distributed systems, Kafka, streaming, cloud, CI/CD, observability, tests, AI agents |
| 27 | Agave | New Grad software engineer | TypeScript, PostgreSQL, Redis, AWS/CDK/EC2, APIs, webhooks, WebSockets, caching, tests |
| 28 | Protocol Nine | Backend Intern | Go/Python, REST, PostgreSQL/Elasticsearch, Kafka, Kubernetes/Docker, cloud, security, observability, RAG |
| 29 | Astranis | Backend Intern | Python, PostgreSQL, RabbitMQ/Flink, Kubernetes, streaming, reliability |
| 30 | Verkada | Backend Intern | Python/Go, PostgreSQL, Redis/DynamoDB, Kafka/SQS, AWS, Docker/Kubernetes, Terraform, REST |
| 31 | OpenAI | Applied SWE Intern | Python, PostgreSQL/MySQL, APIs, reliability, performance |
| 32 | Cloudflare | SWE Intern | Go/Rust/C++/Python/TypeScript, microservices, large-scale networking |
| 33 | Canva | Backend SWE Intern | Java microservices, AWS |
| 34 | Confluent | SWE Intern | Java/Scala/C/C++/Go, Kafka, distributed cloud, tests/deployment |
| 35 | Tessera Labs | Software Engineering Intern, Backend | See §4 |
| 36 | Talos | Backend Intern | Go, PostgreSQL, order-management systems, metrics, low latency |
| 37 | Persona | New Grad SWE | Ruby/Rails, TypeScript, Python, identity/security |
| 38 | Newton Research | Junior Backend + AI | Python Django/DRF, PostgreSQL, Redis, Docker, AWS, GitHub Actions, async, pytest/E2E, Sentry |
| 39 | Baselayer | Junior Backend | Python REST, high-volume data pipelines, security/compliance, testing, monitoring, performance |
| 40 | Asure | Associate SWE | C#/.NET, REST, SQL Server, Terraform, AWS/Azure, Docker, CI/CD, auth/security |
| 41 | Simpson Strong-Tie | Associate Cloud/API SWE | C#/.NET, APIs, Azure, testing, CI/CD |
| 42 | Shrive | Python Backend Entry Level | FastAPI/Django/Flask, PostgreSQL, Redis/Mongo/vector DB, Docker, cloud, queues, OAuth/JWT/OIDC |
| 43 | O2 | Python Backend Entry Level | Python, Lambda, Aurora PostgreSQL/DynamoDB, microservices, APIs, tests, CI/CD |
| 44 | Sentry | New Grad SWE | Python/JavaScript, ingestion/storage/indexing, testing, operations |
| 45 | Harvey | New Grad SWE | Distributed retrieval/AI, secure infrastructure |
| 46 | Valon | New Grad SWE | Distributed systems, regulated finance, reliability/security |
| 47 | Rippling | Backend SWE Intern | Python, platform engineering, scalability |
| 48 | Databricks | New Grad, Distributed Data Systems | Java/C++, query processing, transactions, storage/indexing, performance/security |

**Framework signal (weak):** among second-batch summaries, Django/DRF appears in #38, FastAPI/Django/Flask in #42, and FastAPI + SQLAlchemy in #35. Only #35 had a source check. Do not convert this into a frequency claim.

## 6. Original 24 postings

Original-sample nonexclusive language mentions: Python 12, Go 9, Java 7, TypeScript 7, JavaScript 4, C# 3, C++ 3, C 2, Kotlin 1, Rust 1, Zig 1, Ruby 1. The sample excluded clearly senior roles, duplicates and pre-2025 postings.

Preserved links (historical, **not re-verified**, may be closed):

1. D3 — Backend Engineer, New Grad: https://job-boards.greenhouse.io/d3/jobs/4915295008
2. Peak Games — Software Engineer, Backend, New Grad: https://jobs.lever.co/peakgames/cfaf2251-1472-4e73-b0a3-e3e88df7aa91
3. GoFundMe — Backend Software Engineer Intern (university career-board copy): https://career.ucla.edu/jobs/gofundme-backend-software-engineer-intern/

Remaining identities, URLs not preserved: TikTok (Backend SWE Intern, Media Platform, Summer 2026), AppLovin (Backend Engineer, New Grad), Infinite Computer Solutions (Entry-Level .NET Backend Engineer), Prophet Security (Backend New Graduate), N1 (New Grad Backend Rust), WHOOP (Software Engineer I, Backend), CLEAR (New Grad Backend; removed Jan 2026), Notion (SWE New Grad, Dec 2026), SeatGeek (SWE New Grad 2026), Domino Data Lab (SWE New Grad 2026), Pylon (SWE New Grad/Fall 2025), SentiLink (SWE Full Stack/Backend New Grad), Realm Alliance (SWE New Grad), Samsara (SWE New Grad), SpruceID (Full-Stack SWE New Grad), FurtherAI (Software/AI Engineer New Grad), ATOMS Careers (SWE New Grad), Clara (AI-Native Engineer New Grad), Perpay (SWE New Grad), Zettabyte (SWE New Grad), Tesla (Backend SWE Diagnostics Internship Winter/Spring 2026).

## 7. What the research supports

**Supported (directional only):** both batches heavily emphasize reliability/performance, APIs, cloud, SQL, distributed-system reasoning and tests. The second batch contained more explicitly data-pipeline, observability and AI-oriented roles, possibly due to sampling.

**Not supported:** market prevalence, trends over time, complete source traceability, exact Django/FastAPI frequencies, that listings remain open, or a verified 48-row analysis.

## 8. Project coverage audit

| Competency | NBA Signal Lab coverage | Honest limitation |
|---|---|---|
| Python, REST, OpenAPI | Strong/exact | Django/FastAPI exact frequency unknown |
| PostgreSQL/SQL/ORM/migrations/indexing | Strong | Not equivalent to production DBA experience |
| Data ingestion/pipelines | Strong — core of the project | Portfolio-scale volumes |
| Transactions, consistency, idempotency | Strong | Simulated workload |
| Auth/RBAC/API security | Strong | No real compliance audits |
| Caching | Strong **if measured need** | Redis not installed for keywords |
| Async/background tasks | Strong when backfills warrant | Celery not required in MVP |
| Event streaming/distributed systems | Moderate–strong **if justified and implemented** | Kafka optional; scale simulated |
| Docker/cloud/CI/IaC | Strong when deployed | Real cost and maintenance matter |
| Observability/failure testing | Strong when instrumented | Not a real on-call rotation |
| AI/LLM integration | Not planned for MVP | Optional later; never bolted on |
| Search, GraphQL, gRPC, Kubernetes | Weak/not covered by design | Adding all would be keyword stuffing |
| Java/Spring, Go, .NET, Node-specific | Not exact | Transferable architecture/SQL/API skills only |
| Production ownership, on-call, real users, compliance, large teams | Not demonstrable | Never claim equivalence to employment |

## 9. Next verification steps (optional, not blocking development)

To produce an auditable dataset: retrieve the original research export, or locate and inspect each missing posting; record employer URL, retrieval date/archival status, exact requirements and per-posting coding; mark unavailable listings **unverifiable** rather than substituting different jobs; recode both batches with one frozen rubric before publishing exact frequencies.
