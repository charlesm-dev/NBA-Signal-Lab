# Development Log

Read the **Current state** block at the start of every session. Entries below it are newest first. Record only what actually happened and what was verified.

## Current state

- **Stage:** 2 (Data feasibility spike) — in progress. Task 1 (source research) done.
- **Milestone:** none reached
- **Implemented code:** none
- **Data source:** NBA.com via `nba_api`, private use only, no public demo (ADR-014)
- **Blockers:** none
- **Next task:** Stage 2 Task 2. The developer installs uv, sets up the Python project (Python 3.14, `httpx`/`nba_api`, `pytest`, `ruff`), and records the tooling ADR. Remaining Stage 2 tasks: 3 fetch one game, 4 explore and reconcile the score, 5 offline smoke test with synthetic fixture, 6 write `docs/DATA_SOURCES.md`, then `/verify-stage 2`.

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

### 2026-10-17 — Stage 2, Task 1: source research and decision

**Done:** Claude checked six candidate sources against their primary terms and docs. The developer chose NBA.com via `nba_api` for private use (ADR-014) after confirming the goal is experience and a résumé project, not a public launch.

**Research notes** (input for `docs/DATA_SOURCES.md`; recheck before relying on them):
- **NBA.com Terms of Use** (updated 2026-07-13), Section 9: stats only for "private, non-commercial purposes" with prominent NBA.com attribution. Bans use in products presenting live or archived play-by-play (vi) and in comprehensive stats databases without consent (vii). Section 1 allows downloading to a single computer for personal, noncommercial use.
- **`nba_api`:** MIT-licensed, unofficial client. Third-party reports say stats.nba.com drops requests from cloud IPs (AWS/GCP/Azure) and uses bot protection. Not confirmed for `nba_api` itself.
- **NBA.com live CDN:** returned HTTP 403 to a single plain request from the developer's machine. Not pursued.
- **BALLDONTLIE** (terms 2026-09-17): commercial use and public display allowed; no resale of unmodified data; no competing products. Play-by-play is GOAT tier only ($39.99/mo, 600 req/min) and starts with the 2025-26 season. Substitution coverage unknown. Chosen as the fallback.
- **Sportradar:** 30-day trial (1,000 requests per 30 days, 1 QPS); production for paying customers only.
- **Kaggle:** `wyattowalsh/basketball` (CC BY-SA 4.0, has play-by-play) and `eoinamoore/...` (CC0, box scores) are both NBA.com-derived, so the same private-use limits apply.
- **Basketball-Reference** (terms 2023-05-19): automated access needs written permission. Rejected.

**Still to verify in Tasks 3–4:** play-by-play includes substitutions; event ordering; history depth; request pacing that `nba_api` tolerates from a home connection.

**Note:** the Stage 1 close-out commit `8293c42` had not reached GitHub as of this entry (`git status`: ahead of `origin/main`).

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

**Developer context recorded:** new to Python, Django and FastAPI; strong JavaScript and Git; SQL basics. Target is ~2 months at about 1 hour/day. Public demo wanted if licensing allows (later dropped, ADR-014).

**Open issues**
- Schedule risk: ~50–60 hours makes M1 realistic, M2 a stretch. Revisit after Stage 2.
- Folder name contains spaces (`NBA SIGNAL LAB`). Optionally rename to `nba-signal-lab` before tooling is added. The developer should do this outside a Claude session, since it moves the working directory.
- The handoff's Python version and dependency manager are unresolved. Propose them when the first code lands (Stage 2).
