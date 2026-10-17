# Development Log

Read the **Current state** block at the start of every session. Entries below it are newest first. Record only what actually happened and what was verified.

## Current state

- **Stage:** 1 (Charter and repository) — **complete**
- **Milestone:** none reached
- **Implemented code:** none
- **Verified:** docs structure; repository pushed to GitHub (`charlesm-dev/nba-signal-lab`, `main` in sync with `origin/main`); Stage 2–4 issues opened (developer-reported)
- **Blockers:** none
- **Next task:** Stage 2 (data feasibility spike) — **requires developer approval to start**. Begin with `/stage-kickoff 2`.

## Stage 1 checklist

From [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md) Stage 1, adapted by ADR-013.

- [x] Local git repository initialized (`main`)
- [x] Original handoff documents committed verbatim, then restructured into `docs/`
- [x] Draft charter (`docs/PROJECT_SPEC.md`) with MVP, non-goals and milestones
- [x] ADRs (`docs/DECISIONS.md`), architecture, learning tracker, this log
- [x] `CLAUDE.md` and project skills (`/stage-kickoff`, `/review-my-code`, `/verify-stage`, `/wrap-session`)
- [x] **Developer** reviewed MVP scope and non-goals in `PROJECT_SPEC.md` — accepted as drafted "for now"; revisit after Stage 2 findings
- [x] **Developer** created the GitHub repository and pushed
- [x] **Developer** opened GitHub issues for Stages 2, 3, 4
- [x] **Developer** confirmed handoff content incorporated; `docs/archive/` deleted (originals remain in git history, commit `241bf34`)

## Log

### 2026-10-17 — Stage 1 closed

**Done**
- Developer pushed the repo to GitHub (`charlesm-dev/nba-signal-lab`) over HTTPS with a fine-grained personal access token stored in the macOS Keychain, then opened issues for Stages 2–4.
- Developer accepted `PROJECT_SPEC.md` as drafted; scope to be revisited once Stage 2 shows what data is available.
- Deleted `docs/archive/`. Provenance is now a permalink to commit `241bf34` in `JOB_MARKET_REQUIREMENTS.md`.

**Verified:** `git fetch` shows `main` in sync with `origin/main` before this change. Issue creation is developer-reported and was not checked by Claude (GitHub CLI not installed).

**Open issues**
- The MVP scope was accepted but not yet written in the developer's own words. Before M1, the developer should be able to state the single analysis they would demo.
- Local git author name (`charlesmtcxr-dev`) differs from the GitHub username (`charlesm-dev`). This is harmless if the commit email is linked to the account.

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
