# gsd-phase-planner

**Two-mode planning: discuss strategy first, then create formal plan. Separates exploration from commitment.**

---

## Purpose

Planning fails when you commit before you understand. This skill separates strategy exploration (discuss mode) from tactical commitment (plan mode). Discussion surfaces trade-offs. Planning locks them in.

---

## Triggers

Use when:
- `/gsd discuss-phase N` – explore strategy, no commitment
- `/gsd plan-phase N` – create formal plan with UAT
- "plan the phase," "discuss phase strategy"
- Starting work on a new phase

---

## Two Modes

### Discuss Mode

Interactive exploration:
- Surface trade-offs
- Consider alternatives
- Record decisions to STATE.md
- No files created except STATE.md updates

```
User: /gsd discuss-phase 2

Claude: Phase 2 is Theoretical Framework. Let's discuss approach.

Option A: Start with literature review, then synthesize.
Option B: Start with your thesis, find supporting literature.

Trade-off: A is more rigorous but slower. B risks confirmation bias but ships faster.

What's your priority?
```

### Plan Mode

Formal commitment:
- Creates PLAN.md with task breakdown
- Defines skill mapping per task
- Sets token budget estimates
- Creates UAT.md with acceptance criteria

```
User: /gsd plan-phase 2

Claude: [Creates phases/phase-2/PLAN.md and phases/phase-2/UAT.md]
```

---

## Inputs

- Phase number
- ROADMAP.md (phase objectives)
- STATE.md (prior decisions)
- Discussion outcomes (for plan mode)

---

## Outputs

**Discuss mode:**
- Decisions recorded in STATE.md
- Trade-offs documented
- Strategy alignment confirmed

**Plan mode:**
- `phases/phase-N/PLAN.md` – task breakdown, dependencies, skill mapping
- `phases/phase-N/UAT.md` – acceptance criteria checklist

---

## PLAN.md Structure

```markdown
# Phase N Plan: [Name]

## Task Breakdown

### Task N.1: [Name]
- Objective
- Inputs
- Outputs
- Completion Promise

### Task N.2: ...

## Skill Mapping
| Task | Primary Skill | Supporting Skills |

## Dependencies
[Mermaid diagram]

## Token Budget Estimates
| Task | Input | Output | Total |
```

---

## Common Patterns

**Always discuss first:** Even quick phases benefit from 2 minutes of discussion.

**Revisit during execution:** If you discover the plan is wrong, discuss again before replanning.

**Skip discussion for familiar patterns:** If you've done this type of phase before, go straight to plan.

---

## Notes

- Discussion decisions persist in STATE.md
- Plans reference STATE.md decisions, don't repeat them
- UAT criteria should be verifiable by a human in <5 minutes
- Plans can be revised – but document why in STATE.md
