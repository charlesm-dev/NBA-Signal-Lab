---
name: verify-stage
description: Evidence-based gate check of a roadmap stage or milestone against its definition of done. Runs tests, lint and types, and reports each criterion as verified, not verified, or failed.
argument-hint: "[stage number]"
disable-model-invocation: true
---

# Stage verification

Verify whether stage `$ARGUMENTS` (default: the current stage in `docs/DEVELOPMENT_LOG.md`) meets its definition of done. This is an audit, not an implementation session. Do not fix things here; report them.

## Steps

1. Read the stage's **Done** and **Tests** criteria in `docs/DEVELOPMENT_ROADMAP.md` (plus the milestone definition if this stage ends M1–M5).
2. Run the full check suite listed in the CLAUDE.md Commands section: tests, lint, format check, type check, migrations on a clean database where applicable. Capture the real output.
3. For each criterion, find concrete evidence: a passing test name, command output, a doc section, a measurement file. Read the code when a claim needs checking.
4. Check the hygiene items: no secrets committed (`git grep` for keys/passwords, `.env` not tracked), docs match the implementation, no unrecorded new dependencies, and ADRs exist for decisions the stage required.
5. Note what **cannot** be verified locally and why.

## Output format

```
## Stage N verification — <date>
| Criterion | Status | Evidence |
|---|---|---|
| … | ✅ verified / ❌ failed / ⚠️ not verified | command output, test name, or reason |

**Commands run:** each with its exit status and a short result
**Hygiene:** secrets / docs / dependencies / ADRs
**Gaps blocking completion:** numbered list
**Verdict:** PASS (all criteria verified) / FAIL
```

Only a PASS with no ⚠️ on required criteria counts as done. If this is a milestone PASS, remind the developer to tag the release (e.g. `v0.1.0`). Don't tag it yourself.
