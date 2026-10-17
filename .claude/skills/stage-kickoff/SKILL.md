---
name: stage-kickoff
description: Plan a roadmap stage or task before any implementation. Produces the objective, concepts, files, risks, definition of done, and an implementation objective for the developer to attempt.
argument-hint: "[stage number or task description]"
disable-model-invocation: true
---

# Stage kickoff

Plan the stage or task in `$ARGUMENTS` (if empty, use the "Next task" in the Current state block of `docs/DEVELOPMENT_LOG.md`). **Do not write implementation code in this skill.**

## Steps

1. Read the Current state block of `docs/DEVELOPMENT_LOG.md`, then **only** the matching stage section in `docs/DEVELOPMENT_ROADMAP.md`, including its Amendments section for ordering.
2. Check prerequisites: are the stage's dependencies done according to the log? If not, stop and say what is missing.
3. Inspect only the code and docs this task touches. Check `docs/DECISIONS.md` for relevant ADRs and any PROPOSED decision the stage must settle.
4. Check `docs/LEARNING_PROGRESS.md` to label each concept as **new** or **reinforced**.
5. If the task depends on external data or APIs, list the facts that must be verified first (availability, terms, rate limits, fields). Do not assume them.
6. Break the stage into small tasks. Each one should produce a working, testable improvement in roughly a 1-hour session.

## Output format

```
## Stage N — <name>: <task>
**Objective:** one or two sentences
**Why it matters:** engineering and interview relevance
**Concepts:** new: … | reinforced: …
**Decisions to make:** each with options and your recommendation (ADR needed? yes/no)
**Files/components:** paths to create or change
**Dependencies & risks:** …
**Task breakdown:** numbered small tasks, the first marked ← start here
**Definition of done:** testable criteria and the commands that will verify them
**Your implementation objective:** what the developer writes now, what "working" looks like, hints (not solutions), and what to bring to /review-my-code
```

End by asking for approval or adjustments. Do not proceed past planning.
