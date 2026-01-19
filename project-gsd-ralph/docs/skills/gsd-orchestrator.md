# gsd-orchestrator

**Meta-coordinator routing commands to specialized skills. Single entry point for GSD operations.**

---

## Purpose

Routes `/gsd` commands to the right skill while maintaining awareness of project state. You don't call individual skills directly – you go through the orchestrator.

---

## Triggers

Use when:
- Any `/gsd` command
- Checking project status
- Need to know what to do next
- `/gsd status`, `/gsd help`

---

## Inputs

- Current project state (reads from ORCHESTRATION.md)
- User command (init, plan-phase, execute-phase, verify-phase, etc.)

---

## Outputs

- Routes to appropriate skill
- Returns status summaries
- Suggests next actions when asked

---

## Command Routing

| Command | Routes To |
|---------|-----------|
| `/gsd init` | gsd-init |
| `/gsd discuss-phase N` | gsd-phase-planner (discuss mode) |
| `/gsd plan-phase N` | gsd-phase-planner (plan mode) |
| `/gsd execute-phase N` | gsd-phase-executor |
| `/gsd verify-phase N` | gsd-phase-verifier |
| `/gsd state` | gsd-state-manager |
| `/gsd decide` | gsd-state-manager |
| `/gsd status` | Returns current state |

---

## Example

```
User: /gsd status

Claude: [Reads ORCHESTRATION.md]

Current State:
- Phase: 3 of 6
- Status: Executing
- Last action: Task 3.2 complete
- Next: Task 3.3 (Create handoff schemas)

Suggested action: Continue with `/gsd execute-phase 3` or check progress in PROGRESS.txt
```

---

## Common Patterns

**Status check:** Start sessions with `/gsd status` to see where you left off.

**Navigation:** Use orchestrator to jump between phases or modes.

**Recovery:** If confused about state, orchestrator reads files and tells you where you are.

---

## Notes

- Orchestrator is stateless – all state lives in files
- Reads ORCHESTRATION.md to understand current position
- Won't let you skip phases without good reason
- Maintains the single-entry-point principle
