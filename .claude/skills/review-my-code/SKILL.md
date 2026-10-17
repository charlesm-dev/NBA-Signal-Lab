---
name: review-my-code
description: Learning-oriented senior-engineer review of the developer's own changes. Ranks findings by correctness, security, tests, then design; explains why; checks understanding; proposes evidence-based learning-progress updates.
argument-hint: "[optional: files, commit range, or focus area]"
disable-model-invocation: true
---

# Learning-oriented code review

Review the developer's work as a senior engineer mentoring a junior one. The goal is understanding, not just a correct diff.

## Steps

1. Find the change: `$ARGUMENTS` if given, otherwise `git diff` + `git diff --staged` (and untracked files from `git status`). Read the surrounding code needed to judge it, nothing more.
2. Identify the task and stage from the Current state block of `docs/DEVELOPMENT_LOG.md`, and the stage's definition of done in `docs/DEVELOPMENT_ROADMAP.md`.
3. Run the existing tests, lint and type checks if they are set up (see the CLAUDE.md Commands section). Report results verbatim; never assume a pass.
4. Review in this priority order:
   1. **Correctness:** wrong results, edge cases, data loss, idempotency, transactions, ordering
   2. **Security:** secrets, injection, authz gaps, unsafe input handling, license/rate-limit violations
   3. **Tests:** missing cases, tests that can't fail, mocks where a real DB is required
   4. **Design:** architecture boundaries (CLAUDE.md), naming, abstractions, error handling
   5. **Python idiom:** especially habits carried over from JavaScript
5. Don't rewrite their code. Point to the location (`file:line`), explain the problem and *why* it matters, and give a hint or a minimal illustrative snippet. Give a full fix only if the developer asks or is stuck after a hint.

## Output format

```
**Verdict:** ready / needs changes / blocked
**Checks run:** command → result (or "not run: <reason>")
**Findings:** numbered, highest severity first: severity · file:line · problem · why it matters · hint
**What you did well:** specific, brief
**Concept check:** 2–3 questions that test understanding of the key ideas (e.g. "what happens if this import runs twice?")
**Proposed LEARNING_PROGRESS updates:** concept → level, with evidence (only for what the developer wrote AND explained)
```

Wait for the developer's answers to the concept check before proposing any concept as "Can explain". Only edit `docs/LEARNING_PROGRESS.md` after the developer confirms.
