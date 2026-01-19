# GSD + Ralph Loop Usage Guide

A practical walkthrough for running multi-phase LLM projects without quality degradation.

---

## The Core Loop

Every GSD project follows the same pattern:

```
Initialize → Plan → Execute → Verify → Repeat
```

Initialize once. Then for each phase: plan it, execute it, verify it. Move to the next phase. Files persist everything. Context never accumulates.

---

## Starting a New Project

### 1. Initialize

```
/gsd init
```

The system interviews you:
- **Vision:** What does success look like?
- **Constraints:** Time, resources, technical limits?
- **Success criteria:** How will you know it's done?
- **Phases:** What are the major milestones?

Be brief. Details come during phase planning.

### 2. Review the Generated Structure

```
project/
├── PROJECT.md           # Your answers, formalized
├── REQUIREMENTS.md      # Detailed requirements
├── ROADMAP.md           # Phase overview
├── ORCHESTRATION.md     # Execution state (empty)
└── STATE.md             # Decision log (empty)
```

Skim PROJECT.md and ROADMAP.md. Make sure they capture what you meant.

---

## Planning a Phase

### 1. Discuss First

```
/gsd discuss-phase 1
```

This is exploration, not commitment:
- Surface trade-offs
- Consider alternatives
- Record decisions to STATE.md
- Align on approach

Example exchange:

```
Claude: Phase 1 is Discovery & Research. Two approaches:

A) Breadth-first: Survey everything, then go deep.
B) Depth-first: Pick key areas, go deep immediately.

Trade-off: A finds surprises. B ships faster.

User: A. We don't know what we don't know yet.

Claude: Recorded in STATE.md: "Breadth-first discovery to surface unknowns."
```

### 2. Create the Plan

```
/gsd plan-phase 1
```

This creates two files:

**PLAN.md:**
- Task breakdown
- Skill mapping per task
- Token budget estimates
- Dependencies

**UAT.md:**
- Acceptance criteria checklist
- How you'll verify each deliverable

Review both. Adjust if needed.

---

## Executing a Phase

```
/gsd execute-phase 1
```

The Ralph Loop takes over:

```
Task 1.1: [description]
  → Read bounded context (~3,500 tokens)
  → Execute skill
  → Write compressed handoff
  → TASK_1_1_COMPLETE

Task 1.2: [description]
  → Read bounded context (~3,500 tokens)
  → Execute skill
  → Write compressed handoff
  → TASK_1_2_COMPLETE

...
```

### What Happens Under the Hood

**Read:** Each task loads only what it needs:
- Project summary (300 tokens)
- Current task (500 tokens)
- Previous handoff (500 tokens)
- Skill instructions (1,500 tokens)

**Execute:** The relevant skill does the work.

**Write:** Output compresses to handoff. Progress logs to PROGRESS.txt. State updates in ORCHESTRATION.md.

### Pausing and Resuming

You can stop between any two tasks. All state lives in files.

To resume:
```
/gsd status
```

Shows where you left off. Continue with:
```
/gsd execute-phase N
```

Execution picks up at the next incomplete task.

---

## Verifying a Phase

```
/gsd verify-phase 1
```

This walks through UAT.md:

```
Claude: Criterion 1.1: All GSD skills documented with inputs/outputs.

[Opens SKILL-REGISTRY.md for review]

Does this pass?

User: Yes

Claude: ✅ Passed.

Criterion 1.2: Ralph Loop mechanics documented.

[Opens relevant file]

Does this pass?

User: The compression examples are missing.

Claude: ❌ Failed.

Options:
A) Fix – add compression examples
B) Re-plan – restructure the task
C) Accept with caveat – document and proceed
```

### Handling Failures

**Fix:** Minor issue. Correct it, re-verify.

**Re-plan:** Fundamental problem. Go back to planning.

**Accept with caveat:** Known limitation. Document in STATE.md, proceed.

---

## Common Patterns

### Pattern 1: Research → Synthesis → Output

Typical for content generation, analysis, or documentation projects.

```
Phase 1: Discovery (breadth)
Phase 2: Framework (synthesis)
Phase 3: Gap Analysis (identify missing)
Phase 4: Content Creation (produce output)
Phase 5: Documentation (package)
Phase 6: Review (verify quality)
```

### Pattern 2: Problem Validation

For validating business ideas or product hypotheses.

```
Phase 1: Problem Extraction (clarify the problem)
Phase 2: Market Discovery (who has this problem?)
Phase 3: Awareness Analysis (how do they think about it?)
Phase 4: Report Generation (document findings)
```

### Pattern 3: Multi-Phase Development

For building systems or tools iteratively.

```
Phase 1: Requirements (what are we building?)
Phase 2: Architecture (how will it work?)
Phase 3: Implementation (build it)
Phase 4: Testing (verify it works)
Phase 5: Documentation (explain it)
Phase 6: Deployment (ship it)
```

---

## Working Across Sessions

### Ending a Session

No special action needed. Everything's in files.

### Starting a New Session

```
/gsd status
```

Returns:
```
Project: [Name]
Phase: 3 of 6 (Gap Analysis)
Status: Executing
Last: Task 3.2 complete
Next: Task 3.3

Suggested: /gsd execute-phase 3
```

Continue from where you left off.

### Long Breaks (Days/Weeks)

Same process. Files don't expire. State persists indefinitely.

If you forgot context:
1. Read PROJECT.md (vision)
2. Read STATE.md (decisions)
3. Read ORCHESTRATION.md (current position)
4. Resume with `/gsd execute-phase N`

---

## Troubleshooting

### "I don't know what phase I'm in"

```
/gsd status
```

### "The plan doesn't match what I need"

```
/gsd discuss-phase N
```

Re-explore the strategy. Then:
```
/gsd plan-phase N
```

Create a new plan. Old plan is overwritten.

### "A task keeps failing"

Options:
1. Re-read the task description in PLAN.md
2. Check STATE.md for relevant decisions
3. Simplify the task (break into smaller pieces)
4. Accept partial completion and document why

### "I need to change a decision from a previous phase"

```
/gsd decide
```

Record the new decision. Reference the old one. Document why it changed.

### "Context seems to be accumulating"

This shouldn't happen. Each task reads fresh bounded context.

Check:
- Is PROGRESS.txt logging each task?
- Are handoffs staying under ~800 tokens?
- Is ORCHESTRATION.md updating?

If any are stale, execution may have broken.

### "Human verification is too slow"

UAT criteria should be verifiable in <5 minutes each.

If taking longer:
- Criteria too vague? Make them specific.
- Too many criteria? Consolidate.
- Deliverables hard to find? Organize files better.

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `/gsd init` | Start new project |
| `/gsd status` | Where am I? |
| `/gsd discuss-phase N` | Explore strategy |
| `/gsd plan-phase N` | Create formal plan |
| `/gsd execute-phase N` | Do the work |
| `/gsd verify-phase N` | Human verification |
| `/gsd state` | View decisions |
| `/gsd decide` | Record decision |
| `/gsd question` | Log open question |
| `/gsd tradeoff` | Document trade-off |

---

## Key Files

| File | Purpose |
|------|---------|
| PROJECT.md | Vision, constraints, criteria |
| ROADMAP.md | Phase overview |
| ORCHESTRATION.md | Current execution state |
| STATE.md | Decisions, trade-offs |
| PROGRESS.txt | Execution log |
| phases/phase-N/PLAN.md | Task breakdown |
| phases/phase-N/UAT.md | Acceptance criteria |
| handoffs/phase-N-handoff.json | Compressed inter-phase data |

---

*When in doubt: check files, not memory. That's the whole point.*
