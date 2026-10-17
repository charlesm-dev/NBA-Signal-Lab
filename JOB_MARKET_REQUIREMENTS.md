# NBA Signal Lab — Backend Job-Market Research Handoff

## Evidence and limitations

Previous research described **48 unique early-career backend-oriented postings** from 2025–2026: an original 24 and an additional 24. This document preserves **only the combined counts and original counts actually recoverable from the prior analysis**, rather than reconstructing unprovided individual records. The full second-sample company-by-company coding, complete 48 posting URLs and exact framework counts are **not available in the supplied conversation record**. Consequently this is a faithful *summary of prior research*, **not a fresh independent verification or a reproducible 48-row audit**. Do not manufacture missing counts. Original sample deliberately excluded clearly senior roles, duplicates, and pre-2025 postings; percentages are explicit-mention rates in a nonrandom, convenience sample, not market prevalence or statistically representative estimates.

## Normalization

- Backend services/APIs: REST, GraphQL, gRPC and backend endpoint development (but keep protocol-specific mentions separately).
- SQL/relational: PostgreSQL, MySQL, MariaDB, SQL Server, Oracle and SQL schema/ORM.
- Cloud: AWS, Azure, GCP, etc. Containers: Docker/Kubernetes (orchestration distinct where relevant).
- Infrastructure/deployment: IaC, Terraform, ECS/Kubernetes and deployment tooling; CI/CD separately.
- Async/event-driven: Kafka, RabbitMQ, SQS, pub/sub, background processing; distinguish task queues vs durable streams.
- Caching: Redis, Memcached and cache design.
- Testing: unit/integration/automated tests. Security: authentication, authorization, privacy, API security.
- Distributed systems: explicit distributed-system/service coordination. Scale/reliability/performance: explicitly stated expectations.
- Count each posting at most once per competency; different competencies can overlap. Language mentions are nonexclusive.

## Recoverable competency matrix

| Competency | Original 24 | Combined 48 | Delta, original to combined | Classification and project relevance |
|---|---:|---:|---|---|
| Scalability/reliability/performance | 21/24 (87.5%) | **44/48 (91.7%)** | +4.2 pp | Essential; load testing, replay recovery, latency SLOs |
| Backend APIs/services | 21/24 (87.5%) | **37/48 (77.1%)** | -10.4 pp | Essential; FastAPI/DRF, contracts, pagination |
| Cloud | 16/24 (66.7%) | **32/48 (66.7%)** | 0 pp | Essential/high-value; AWS deployment/IAM |
| Distributed systems | 14/24 (58.3%) | **30/48 (62.5%)** | +4.2 pp | High-value; worker recovery, messaging and projections |
| SQL/relational | 16/24 (66.7%) | **29/48 (60.4%)** | -6.3 pp | Essential/high-value; PostgreSQL schema, SQL tuning |
| Testing | 12/24 (50.0%) | **26/48 (54.2%)** | +4.2 pp | High-value; golden-game, DB, API, failure tests |
| Python language | 12/24 (50.0%) | **25/48 (52.1%)** | +2.1 pp | Strong exact language overlap |
| Containers | 16/24 (66.7%) | **22/48 (45.8%)** | -20.9 pp | High-value; Docker/Compose; no automatic K8s |
| Messaging/event-driven | 8/24 (33.3%) | **17/48 (35.4%)** | +2.1 pp | Useful/common in relevant roles; Celery/RabbitMQ, Kafka only if justified |
| Caching | 5/24 (20.8%) | **10/48 (20.8%)** | 0 pp | Useful; Redis where measured |
| CI/CD | 5/24 (20.8%) | **10/48 (20.8%)** | 0 pp | Useful, professionally expected; Actions |
| Infrastructure/IaC/deployment | 10/24 (41.7%) | **not recoverable** | unknown | High-value, AWS/Terraform if justified |
| GraphQL specifically | 4/24 (16.7%) | **not recoverable** | unknown | Specialized; not necessary for NBA Signal Lab |
| Security/auth explicitly | 5/24 (20.8%) | **not recoverable** | qualitative increase in second sample reported, no verified count | Essential engineering standard despite job-ad undermention |
| Observability | original exact count not preserved | **not recoverable** | qualitative increase reported | High-value; tracing/metrics/logs |
| Data ingestion/pipelines | original exact count not preserved | **not recoverable** | identified as notable in second sample | Especially strong fit for NBA Signal Lab |
| Async/background processing | not separately coded | **not recoverable** | recurring theme | Celery and async provider I/O |
| File/object storage | not separately coded | **not recoverable** | relevant in cloud/data roles | Optional S3 raw-data archive |
| Concurrency/idempotency | not separately coded | **not recoverable** | implicit in distributed systems | Strong via imports/replay/worker duplicates |
| API-specific gRPC, OpenAPI, REST | not fully separately coded | **not recoverable** | unknown | REST/OpenAPI directly; gRPC not needed |
| Search engines | not fully separately coded | **not recoverable** | unknown | PostgreSQL search first; external search engine unjustified |

**Important:** A prior narrative described “48-posting dataset” but did not preserve a complete row-by-row ledger here. To calculate exact new-only counts for a competency with a combined and original count, subtract original count from combined count; e.g. API = 16/24 (66.7%), SQL = 13/24 (54.2%), cloud = 16/24 (66.7%), scale = 23/24 (95.8%), distributed = 16/24 (66.7%), testing = 14/24 (58.3%), Python = 13/24 (54.2%), containers = 6/24 (25.0%), messaging = 9/24 (37.5%), caching = 5/24 (20.8%), CI/CD = 5/24 (20.8%). These are arithmetic consequences of **previously reported** counts, not independent new audits.

## Language and ecosystem observations

Original 24, nonexclusive mentions: Python 12, Go 9, Java 7, TypeScript 7, JavaScript 4, C# 3, C++ 3, C 2, Kotlin 1, Rust 1, Zig 1, Ruby 1 (Node.js runtime was separately noted once in a manual tally, not a language). Combined Python **25/48**. **Exact combined Java/Spring, Django and FastAPI mention splits are not recoverable here; do not infer them.** The earlier discussion favored Python + Django + FastAPI for the user’s project, while recognizing Java/Spring, Go, Node/TypeScript and C#/.NET as valid ecosystems. A Python project demonstrates transferable competencies but does **not** literally satisfy Java/Spring, Go or .NET-specific experience requirements.

## Original 24 postings — identifiable supporting sources

These were the postings cited in the earlier analysis. Links are historical research references, **not claims that listings are currently open**. Some original citations were to third-party mirrors or search results; recheck dates, requirements and canonical pages before relying on them for a fresh analysis.

1. D3 — Backend Engineer, New Grad: https://job-boards.greenhouse.io/d3/jobs/4915295008
2. Peak Games — Software Engineer, Backend, New Grad: https://jobs.lever.co/peakgames/cfaf2251-1472-4e73-b0a3-e3e88df7aa91
3. GoFundMe — Backend Software Engineer Intern: https://career.ucla.edu/jobs/gofundme-backend-software-engineer-intern/
4. TikTok — Backend Software Engineer Intern, Media Platform, Summer 2026: earlier Indeed search citation; stable URL not preserved.
5. AppLovin — Backend Engineer, New Grad: earlier Employbl search citation; stable URL not preserved.
6. Infinite Computer Solutions — Entry-Level .NET Backend Engineer: earlier UCLA Career Center search citation; stable URL not preserved.
7. Prophet Security — Backend New Graduate: earlier Ashby citation; URL not preserved.
8. N1 — New Grad Backend Rust: earlier Ashby citation; URL not preserved.
9. WHOOP — Software Engineer I, Backend: earlier Ashby citation; URL not preserved.
10. CLEAR — New Grad Backend: earlier listing (removed Jan 2026); URL not preserved.
11. Notion — Software Engineer New Grad, Dec 2026: earlier listing; URL not preserved.
12. SeatGeek — Software Engineer New Grad, 2026: earlier listing; URL not preserved.
13. Domino Data Lab — Software Engineer New Grad 2026: earlier listing; URL not preserved.
14. Pylon — Software Engineer New Grad/Fall 2025: earlier listing; URL not preserved.
15. SentiLink — Software Engineer Full Stack/Backend New Grad: earlier listing; URL not preserved.
16. Realm Alliance — Software Engineer New Grad: earlier listing; URL not preserved.
17. Samsara — Software Engineer New Grad: earlier listing; URL not preserved.
18. SpruceID — Full-Stack Software Engineer New Grad: earlier listing; URL not preserved.
19. FurtherAI — Software/AI Engineer New Grad: earlier listing; URL not preserved.
20. ATOMS Careers — Software Engineer New Grad: earlier listing; URL not preserved.
21. Clara — AI-Native Engineer New Grad: earlier listing; URL not preserved.
22. Perpay — Software Engineer New Grad: earlier listing; URL not preserved.
23. Zettabyte — Software Engineer New Grad: earlier listing updated Sep 2026; URL not preserved.
24. Tesla — Backend Software Engineer Diagnostics Internship Winter/Spring 2026: earlier listing; URL not preserved.

**Additional 24 posting identities and source URLs are missing from the accessible prior record.** Claude should request the previous research export or redo a separately labeled verification pass if a fully auditable 48-row dataset is required. Do not invent identities or pretend this document contains all 48 individual sources.

## Project-coverage audit

| Competency | NBA Signal Lab coverage | Honest limitation |
|---|---|---|
| Python, REST, OpenAPI | Strong/exact | Django/FastAPI exact framework frequency unknown |
| PostgreSQL/SQL/ORM/migrations/indexing | Strong | Not equivalent to years of production DBA work |
| Transactions, consistency, idempotency | Strong | Simulated workload, not actual league-scale operation |
| Auth/RBAC/API security | Strong | Real-world compliance audits not demonstrated |
| Caching | Strong **if measured need** | Redis should not be installed merely for keywords |
| Async/background tasks | Strong when backfills warrant | Celery not required in MVP |
| Event streaming/distributed systems | Moderate–strong **if justified and implemented** | Kafka optional; real scale only simulated |
| Docker/cloud/CI/IaC | Strong when deployed | Actual cloud cost and maintenance matter |
| Observability/failure testing | Strong when instrumented | Personal testing not actual on-call rotation |
| Search, GraphQL, gRPC, Kubernetes | Weak/not covered by design | Equivalent API/infra concepts; adding all would be keyword stuffing |
| Java/Spring, Go, .NET, Node-specific | Not exact | Transferable architecture/SQL/API skills only |
| Multi-year production ownership, on-call, real users, enterprise compliance, large-team collaboration | Not project-demonstrable | Must not claim equivalence to employment |

## Research interpretation

The expanded counts support the same broad thesis: **API engineering, reliability/performance, SQL, cloud, testing and distributed-system fundamentals** remain valuable. Containers appeared less often in the second sample; this does not mean Docker is unimportant. Data pipelines/async/observability became more salient qualitatively but lack recoverable exact counts. Job descriptions understate cross-cutting practices. Avoid chasing a rare tool unless NBA Signal Lab genuinely needs it. Reproducibility requires a 48-row source ledger, a frozen coding rubric and dated copies of each posting.
