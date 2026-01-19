# gsd-phase-verifier

**Guide UAT for completed phases. Human verification at quality gates.**

---

## Purpose

AI can't reliably self-assess quality. This skill facilitates human verification at phase boundaries. Presents acceptance criteria, guides the review, handles pass/fail outcomes. Without verification, errors compound.

---

## Triggers

Use when:
- `/gsd verify-phase N`
- "verify phase," "check phase completion," "UAT"
- Phase execution is complete
- Before moving to next phase

---

## The Verification Process

```
1. Load UAT.md for the phase
2. Present criteria one by one
3. Human marks pass/fail for each
4. If all pass → phase complete
5. If any fail → options: fix, re-plan, or accept with caveat
```

---

## Inputs

- `phases/phase-N/UAT.md` – acceptance criteria
- Phase deliverables (files, outputs)
- Human judgment

---

## Outputs

- Completed UAT checklist
- Pass/fail decision
- If failed: documented issues and chosen resolution
- If passed: phase marked complete in ORCHESTRATION.md

---

## Example

```
User: /gsd verify-phase 3

Claude: Let's verify Phase 3: Gap Analysis & Skill Development

Criterion 3.1: Skill registry exists with all 17 skills documented.
[Opens SKILL-REGISTRY.md for review]

Does this pass?

User: Yes

Claude: ✅ Criterion 3.1 passed.

Criterion 3.2: Dependency diagrams show all skill relationships.
[Opens SKILL-DEPENDENCIES.md]

Does this pass?

User: The gsd-state-manager connections are missing.

Claude: ❌ Criterion 3.2 failed.

Options:
A) Fix now – add the missing connections
B) Re-plan – revise the task and re-execute
C) Accept with caveat – document the gap and proceed

Which approach?
```

---

## Handling Failures

**Fix:** Minor issues. Correct them, re-verify the criterion.

**Re-plan:** Fundamental misunderstanding. Go back to planning.

**Accept with caveat:** Known limitation. Document in STATE.md, proceed anyway.

---

## UAT.md Structure

```markdown
# Phase N UAT: [Name]

## Acceptance Criteria

### N.1 [Category]
- [ ] Criterion N.1.1: [Specific, verifiable statement]
- [ ] Criterion N.1.2: ...

### N.2 [Category]
- [ ] Criterion N.2.1: ...

## Overall Phase Criteria
- [ ] Quality criteria that span multiple deliverables

## Phase Completion
All criteria must pass for: `PHASE_N_COMPLETE`
```

---

## Common Patterns

**Quick verification:** Simple phases with clear deliverables. 5-minute review.

**Deep verification:** Complex phases. Review each deliverable thoroughly.

**Partial pass:** Some criteria pass, some fail. Fix the failures or document why they're acceptable.

---

## Notes

- Human judgment is required – this isn't automated
- Criteria should be verifiable in <5 minutes each
- "Pass with caveat" is valid but must be documented
- Never skip verification – that's how errors compound
