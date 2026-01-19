# gsd-init

**Initialize projects through structured interview. Creates canonical file structure.**

---

## Purpose

Gathers vision, constraints, and success criteria through conversation, then generates the foundational files every GSD project needs. Without proper initialization, projects drift.

---

## Triggers

Use when:
- Starting any new project
- `/gsd init`
- "create project," "initialize project," "bootstrap project"
- Beginning a Get Shit Done workflow

---

## Inputs

Interview responses covering:
- **Vision:** What success looks like
- **Constraints:** Time, resources, technical limits
- **Success Criteria:** How you'll know it's done
- **Phase Structure:** Major milestones

---

## Outputs

```
project/
├── PROJECT.md           # Vision, constraints, success criteria
├── REQUIREMENTS.md      # Detailed requirements
├── ROADMAP.md           # Phase overview with estimates
├── ORCHESTRATION.md     # Execution state (starts empty)
└── STATE.md             # Decision log (starts empty)
```

---

## Example

```
User: /gsd init

Claude: Let's initialize your project. What's the vision – what does success look like?

User: Build a research synthesis system that maintains quality across 10+ phases.

Claude: What constraints are we working with?

User: Two weeks, single developer, must work with any LLM provider.

Claude: How will we know it's done?

User: Paper published, system documented, examples work.

Claude: [Creates PROJECT.md, REQUIREMENTS.md, ROADMAP.md, ORCHESTRATION.md, STATE.md]
```

---

## Common Patterns

**Minimal init:** Answer only required questions. Fill in details during phase planning.

**Detailed init:** Thorough interview upfront. Better for complex projects where requirements are known.

**Re-init:** If project scope changes significantly, run init again and merge with existing files.

---

## Notes

- Always run init before other GSD commands
- The interview can be brief – phase planning adds detail later
- PROJECT.md is the source of truth for vision
- ROADMAP.md can be adjusted as you learn
