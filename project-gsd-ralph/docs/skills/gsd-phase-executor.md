# gsd-phase-executor

**Ralph Loop implementation. Bounded iterations, compressed handoffs, completion promises.**

---

## Purpose

Executes phase plans through bounded context iterations. Each iteration: read ~3,500 tokens, execute one task, write compressed output. This is where the actual work happens – within strict constraints that prevent quality degradation.

---

## Triggers

Use when:
- `/gsd execute-phase N`
- "execute the plan," "run phase," "start execution"
- Phase is planned and ready for work

---

## The Ralph Loop

```
For each task in PLAN.md:
  1. READ bounded context:
     - Project summary (~300 tokens)
     - Current task (~500 tokens)
     - Previous handoff (~500 tokens)
     - Skill instructions (~1,500 tokens)
     Total: ~3,300 tokens

  2. EXECUTE single skill:
     - Do the work
     - Generate output with completion promise

  3. WRITE compressed state:
     - Append to PROGRESS.txt
     - Write/update handoff
     - Update ORCHESTRATION.md
```

---

## Token Budget

| Component | Target | Maximum |
|-----------|--------|---------|
| Project summary | 300 | 400 |
| Task description | 500 | 700 |
| Previous handoff | 500 | 800 |
| Skill instructions | 1,500 | 2,000 |
| Buffer | 700 | – |
| **Total** | **3,500** | **3,900** |

Component exceeds budget? Compress before loading.

---

## Inputs

- `phases/phase-N/PLAN.md` – task breakdown
- `ORCHESTRATION.md` – current position
- `handoffs/` – previous phase handoff
- Skill files as specified in plan

---

## Outputs

- Task outputs (files, research, drafts, etc.)
- `PROGRESS.txt` updates (timestamped)
- `handoffs/phase-N-handoff.json` (compressed)
- `ORCHESTRATION.md` updates (state changes)

---

## Completion Promises

Every task defines a completion promise – a specific string that appears only when genuinely done:

```
RESEARCH_COMPLETE
DRAFT_COMPLETE
SCHEMA_VALIDATED
```

Progress is machine-verifiable:
```
Task complete iff Promise ∈ Output
```

---

## Example

```
User: /gsd execute-phase 2

Claude: [Reads PLAN.md – sees 5 tasks]

Starting Task 2.1: Literature Review

[Reads bounded context: project summary, task, previous handoff, skill instructions]

[Executes research-analyst skill]

[Writes to /research/LITERATURE-REVIEW.md]

Task 2.1 complete: LITERATURE_REVIEW_COMPLETE

Starting Task 2.2: Theoretical Framework
...
```

---

## Compression Rules

When writing handoffs:

- **Extract conclusions**, not reasoning chains
- **Include data**, not process descriptions
- **Use pointers** (file paths) instead of payloads (file contents)
- **Maintain structure** via JSON schema
- **Preserve completion promise** as required field

10,000-token output → 500-1,000 token handoff (5-10x compression)

---

## Common Patterns

**Pause and resume:** Execution can stop between any two tasks. State lives in files.

**Skill switching:** Different tasks use different skills. Executor handles the transitions.

**Error recovery:** If a task fails, executor logs the failure and suggests recovery options.

---

## Notes

- Never accumulates context across tasks
- Each task starts fresh with bounded input
- Quality stays constant regardless of phase depth
- Human can intervene between any two tasks
