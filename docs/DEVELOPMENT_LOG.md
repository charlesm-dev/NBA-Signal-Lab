# Development Log

Read the **Current state** block at the start of every session. Entries below it are newest first. Record only what actually happened and what was verified.

## Current state

- **Stage:** 1 (Charter and repository) — in progress
- **Milestone:** none reached
- **Implemented code:** none
- **Verified:** documentation structure only; no code, tests or tooling exist
- **Blockers:** none
- **Next task:** the developer reviews `docs/PROJECT_SPEC.md`, edits MVP scope and non-goals in their own words, and commits. Then: create the GitHub repository, push, and open issues for Stages 2–4.
- **After Stage 1:** Stage 2 (data feasibility spike) — **requires developer approval to start**.

## Stage 1 checklist

From [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md) Stage 1, adapted by ADR-013.

- [x] Local git repository initialized (`main`)
- [x] Original handoff documents committed verbatim, then restructured into `docs/`
- [x] Draft charter (`docs/PROJECT_SPEC.md`) with MVP, non-goals and milestones
- [x] ADRs (`docs/DECISIONS.md`), architecture, learning tracker, this log
- [x] `CLAUDE.md` and project skills (`/stage-kickoff`, `/review-my-code`, `/verify-stage`, `/wrap-session`)
- [ ] **Developer** reviews and finalizes MVP scope and non-goals in `PROJECT_SPEC.md`
- [ ] **Developer** creates the GitHub repository and pushes
- [ ] **Developer** opens GitHub issues for the next three stages (2, 3, 4)
- [ ] **Developer** confirms the handoff content is incorporated, so `docs/archive/` can be removed (or kept deliberately)

## Log

### 2026-10-16 — Project setup (Stage 1)

**Done**
- Reviewed the planning handoff, roadmap, job-market research and the second-batch research audit.
- Agreed four roadmap/architecture clarifications (ADR-010 to ADR-013): ingestion as a package + CLI; Postgres-only Compose from Stage 3 with early basic CI; Django after M1; documentation structure.
- Created `docs/` structure, `CLAUDE.md`, four project skills and `.gitignore`. Original files were committed first for provenance, then moved (`docs/archive/` holds the handoff and second-batch audit).
- Merged second-batch research into `JOB_MARKET_REQUIREMENTS.md`, keeping the verified/unverified distinction. Rechecking Tessera Labs (#35): the posting ID is no longer on the employer's public Ashby board, so Claude could not independently confirm it.

**Decisions:** ADR-010, ADR-011, ADR-012, ADR-013.

**Developer context recorded:** new to Python, Django and FastAPI; strong JavaScript and Git; SQL basics. Target is ~2 months at about 1 hour/day. Public demo wanted if licensing allows.

**Open issues**
- Schedule risk: ~50–60 hours makes M1 realistic, M2 a stretch. Revisit after Stage 2.
- Folder name contains spaces (`NBA SIGNAL LAB`). Optionally rename to `nba-signal-lab` before tooling is added. The developer should do this outside a Claude session, since it moves the working directory.
- The handoff's Python version and dependency manager are unresolved. Propose them when the first code lands (Stage 2).
