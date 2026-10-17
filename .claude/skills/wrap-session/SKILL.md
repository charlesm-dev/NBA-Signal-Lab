---
name: wrap-session
description: End-of-session context preservation. Updates the development log's Current state and history, records decisions and learning evidence, and names the next concrete task.
disable-model-invocation: true
---

# Wrap session

Preserve context so the next session can start from `docs/DEVELOPMENT_LOG.md` alone. Edit only what changed this session. Don't rewrite unrelated docs.

## Steps

1. Establish what happened: review this conversation, `git status`, and `git log` since the last log entry.
2. **`docs/DEVELOPMENT_LOG.md`:**
   - Rewrite the **Current state** block: stage, milestone, what is implemented, what is verified (with how), blockers, the next concrete task (small enough for one ~1-hour session), and any approval needed.
   - Tick completed items on the stage checklist.
   - Add a dated entry at the top of the Log, covering: done, verified (commands and results), decisions, open issues. Facts only; anything unverified is labeled as such.
3. **`docs/DECISIONS.md`:** if a significant decision was made, add an ADR (and a row in the index table). If a decision was discussed but not made, list it as an open issue in the log instead.
4. **`docs/LEARNING_PROGRESS.md`:** propose level changes with evidence and add a dated session note. Apply only the changes the developer confirms.
5. If `CLAUDE.md` Commands or Conventions changed in reality (new tool, new command), update that section.
6. Show the developer a short summary of every doc change. Ask whether to commit the doc updates, and commit only if they say yes.
