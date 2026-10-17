# NBA Signal Lab

A backend-first NBA event-data platform: it ingests permitted NBA data, normalizes it into canonical PostgreSQL records, deterministically reconstructs game state (possessions), and serves historical game-state analytics through APIs.

**source data → normalized event history → derived basketball state → queryable historical intelligence**

> **Status:** Stage 1 — project charter and planning. No application code yet. See [docs/DEVELOPMENT_LOG.md](docs/DEVELOPMENT_LOG.md).

## Stack

Python · Django + DRF (control plane: users, workspaces, saved analyses) · FastAPI + Pydantic (NBA data and analytics API) · PostgreSQL

Additional infrastructure (Celery, Redis, Kafka, AWS) is added only after a measured need. See [ADR-008](docs/DECISIONS.md#adr-008--infrastructure-admitted-only-on-measured-need).

## Documentation

| Document | Contents |
|---|---|
| [Project spec](docs/PROJECT_SPEC.md) | Vision, MVP, non-goals, milestones |
| [Architecture](docs/ARCHITECTURE.md) | Components, boundaries, data flow, standards |
| [Roadmap](docs/DEVELOPMENT_ROADMAP.md) | 21 ordered stages with acceptance criteria |
| [Decisions](docs/DECISIONS.md) | Architecture decision records |
| [Development log](docs/DEVELOPMENT_LOG.md) | Current state and history |
| [Job-market requirements](docs/JOB_MARKET_REQUIREMENTS.md) | Competencies the project targets, with evidence caveats |
| [Learning progress](docs/LEARNING_PROGRESS.md) | Concepts learned while building |

## Data

No NBA data source has been selected yet. Source licensing and feasibility are evaluated in Stage 2 before any schema work. This project does not claim access to licensed live NBA data.
