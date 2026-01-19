# gsd-state-manager

**Maintain STATE.md for decisions, trade-offs, assumptions, and learnings. Institutional memory that survives context resets.**

---

## Purpose

Decisions get made. Trade-offs get accepted. Assumptions get recorded. Without a system to track these, you redecide the same things repeatedly or forget why you chose what you chose. This skill maintains that institutional memory.

---

## Triggers

Use when:
- `/gsd state` – view current state
- `/gsd decide` – record a decision
- `/gsd question` – log an open question
- `/gsd tradeoff` – document a trade-off
- "record decision," "track trade-off," "open question"

---

## Modes

### View Mode

Display current STATE.md contents:
```
User: /gsd state

Claude: Current project state:

DECISIONS:
- D1: Use file-based persistence over database (simplicity > scalability)
- D2: 3,500 token budget per iteration (conservative, can increase later)

TRADE-OFFS:
- T1: Sequential execution only (parallel deferred to future work)

OPEN QUESTIONS:
- Q1: Should handoff schemas auto-generate?

LEARNINGS:
- L1: Compression ratios consistently 10-15x for research phases
```

### Decision Mode

Record a new decision:
```
User: /gsd decide

Claude: What decision was made?

User: We'll use JSON schemas for all handoffs.

Claude: What was the reasoning?

User: Unstructured handoffs drift. Schemas enforce completeness.

Claude: [Adds to STATE.md under DECISIONS]

Recorded: "D3: JSON schemas for all handoffs (prevents drift, enforces completeness)"
```

### Question Mode

Log an open question:
```
User: /gsd question

Claude: What question needs tracking?

User: How should we handle parallel branch execution?

Claude: [Adds to STATE.md under OPEN QUESTIONS]

Recorded: "Q2: How should we handle parallel branch execution?"
```

### Trade-off Mode

Document a trade-off:
```
User: /gsd tradeoff

Claude: What are you trading off?

User: We're accepting manual schema definition instead of auto-generation.

Claude: What's gained? What's lost?

User: Gained: Ships now. Lost: Ongoing manual effort.

Claude: [Adds to STATE.md under TRADE-OFFS]

Recorded: "T2: Manual schema definition (ships now vs. auto-generation later)"
```

---

## Inputs

- User input describing decision/question/trade-off
- Current STATE.md contents

---

## Outputs

- Updated STATE.md
- Confirmation of what was recorded

---

## STATE.md Structure

```markdown
# Project State

## Decisions
| ID | Decision | Reasoning | Date |
|----|----------|-----------|------|
| D1 | ... | ... | ... |

## Trade-offs
| ID | What | Gained | Lost | Date |
|----|------|--------|------|------|
| T1 | ... | ... | ... | ... |

## Open Questions
| ID | Question | Status | Date |
|----|----------|--------|------|
| Q1 | ... | Open | ... |

## Assumptions
| ID | Assumption | Validated? | Date |
|----|------------|------------|------|
| A1 | ... | No | ... |

## Learnings
| ID | Learning | Source | Date |
|----|----------|--------|------|
| L1 | ... | Phase 2 | ... |
```

---

## Common Patterns

**Pre-planning review:** Before planning a phase, review STATE.md for relevant decisions.

**Post-phase learning:** After completing a phase, record what you learned.

**Decision archaeology:** When confused about past choices, check STATE.md.

---

## Notes

- STATE.md is the source of truth for "why we did X"
- Decisions can be revisited but should be documented when changed
- Open questions should close eventually – track resolution
- Trade-offs are honest acknowledgments, not failures
