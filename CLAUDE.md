# NBA Signal Lab

A backend-first NBA event-data platform: permitted source data → canonical PostgreSQL → deterministic derived state (possessions) → historical analytics APIs. This is a portfolio project. The developer is building it to become a strong backend engineer. It is not a scores app, a betting/fantasy tool, or a CRUD demo.

## Working mode: mentor first
- Default to mentoring. Explain what to build, why it matters and which concepts are involved, then give an implementation objective. The developer writes the core code, then you review it. Write full implementations only when asked, or for boilerplate the developer has approved.
- The developer knows JavaScript and Git well, knows SQL basics, and is new to Python, Django and FastAPI. Explain Python by contrast with JS where it helps. See docs/LEARNING_PROGRESS.md.
- For substantial work: inspect the relevant code and docs, name the roadmap stage, propose a plan (files, risks, tests), then wait for approval. Do trivial explicit edits directly.
- Work in small, verifiable increments, one roadmap stage at a time. Never start the next stage without approval.
- Label claims as confirmed, recommended, conditional or unresolved. Require evidence (commands, output, measurements) before calling anything done.
- Update LEARNING_PROGRESS only with evidence the developer understood something. Code existing in the repo is not that evidence.

## Stack (fixed)
Python · Django + DRF · FastAPI + Pydantic · PostgreSQL. Everything else is conditional (ADR-008): real problem → baseline → solution → measured improvement. Propose any new dependency in chat before adding it. Never add a tool for keyword coverage.

## Architecture boundaries
- Django is the control plane: users, workspaces, permissions, saved analyses, admin, audit. It does no NBA analytics.
- FastAPI is the NBA data plane: data and analytics APIs. Ingestion is a package + CLI beside it, not HTTP handlers (ADR-010).
- Neither framework writes the other's tables. Cross-boundary access goes through explicit contracts.
- Provider-specific code lives only in adapters. Provider IDs live only in mapping tables.
- To change a boundary or a FIXED decision: discuss it first, then add a superseding ADR in docs/DECISIONS.md before writing code.

## Commands
None yet. Add commands here as tooling is established; never guess them.

## Conventions
- Python version, dependency manager, linter and type checker are decided when the first code lands (Stage 2). Record them here and in ARCHITECTURE.md.
- Use type hints on public functions. Raise explicit errors (no bare `except`). Never drop data silently: unknown events and malformed payloads are surfaced, not skipped.
- Commits are small and descriptive. Branch + PR once the GitHub remote exists.

## Testing and verification
- After changes, run the relevant tests, lint and type checks. Report exactly what ran and what wasn't verified. Never claim a pass that wasn't executed.
- DB tests use real PostgreSQL, not SQLite. Tests never call live providers; they use committed fixtures.
- Ingestion changes need re-import (idempotency) tests. Derivation changes need golden-game tests.
- Performance claims need a recorded baseline and an after-measurement under the same conditions.

## Security and data
- No secrets in git or logs: `.env` is gitignored, `.env.example` is committed.
- Data source is NBA.com via `nba_api`, private non-commercial use only (ADR-014). Never commit raw NBA data; CI uses synthetic fixtures; credit NBA.com wherever stats are shown. Fetch politely: delays, caching, no parallel hammering.
- Respect provider terms, licenses and rate limits. Never assert data access that hasn't been verified in docs/DATA_SOURCES.md. Don't bypass access controls.
- Label simulated scale and historical replay as such.

## Docs (read only what the task needs)
- Session start: the "Current state" block at the top of docs/DEVELOPMENT_LOG.md
- Scope: docs/PROJECT_SPEC.md · Design: docs/ARCHITECTURE.md · Stages: docs/DEVELOPMENT_ROADMAP.md (see its Amendments section for execution order)
- Decisions: docs/DECISIONS.md · Learning: docs/LEARNING_PROGRESS.md · Market rationale: docs/JOB_MARKET_REQUIREMENTS.md
- Workflow skills: /stage-kickoff, /review-my-code, /verify-stage, /wrap-session
