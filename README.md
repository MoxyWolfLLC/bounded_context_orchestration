# GSD + Ralph Loop

**Bounded Context Orchestration for LLM Workflows That Don't Fall Apart**

---

## What This Is

A methodology for running complex, multi-phase AI workflows without the quality degradation that kills most long-running LLM projects. Six orchestration skills. One execution pattern. Seven design principles. No vector databases required.

---

## The Problem It Solves

String six LLM tasks together and watch what happens. Phase 1 outputs beautifully. Phase 2 builds on it. By phase 4, the model's forgotten what it decided in phase 2. Context accumulates – 4K becomes 12K becomes 39K – and attention scatters. The context window becomes a graveyard of forgotten decisions.

Most solutions throw more capacity at this: longer windows, RAG, memory systems. They address *what* to include. Not *how much* accumulates.

GSD + Ralph Loop takes a different approach: **treat the LLM as a stateless function with hard input limits.** Every iteration stays under 3,500 tokens. State lives in files. Outputs compress to schema-enforced handoffs. Phase 50 operates with the same attention quality as phase 1.

---

## Quick Start (5 Minutes)

### 1. Initialize a Project

```
/gsd init
```

Answer the interview questions about your project's vision, constraints, and success criteria. GSD creates:

```
project/
├── PROJECT.md           # Vision and constraints
├── REQUIREMENTS.md      # Detailed requirements
├── ROADMAP.md           # Phase overview
├── ORCHESTRATION.md     # Execution state
└── STATE.md             # Decisions and trade-offs
```

### 2. Plan Your First Phase

```
/gsd discuss-phase 1
```

Explore strategy. Surface trade-offs. Record decisions. Then:

```
/gsd plan-phase 1
```

Creates `PLAN.md` with task breakdown, skill mapping, and UAT criteria.

### 3. Execute

```
/gsd execute-phase 1
```

Ralph Loop takes over: bounded iterations, compressed handoffs, completion promises.

### 4. Verify

```
/gsd verify-phase 1
```

Walk through UAT criteria. Human confirms. Phase completes. Move to phase 2.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                ORCHESTRATION LAYER                   │
│  gsd-init → gsd-orchestrator → gsd-phase-planner   │
│         → gsd-phase-executor → gsd-phase-verifier   │
└─────────────────────────┬───────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                 EXECUTION LAYER                      │
│           Ralph Loop: Read → Execute → Write         │
│        (~3,500 tokens per iteration, always)         │
└─────────────────────────┬───────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                   STATE LAYER                        │
│   PROJECT.md | STATE.md | PROGRESS.txt | handoffs/  │
└─────────────────────────────────────────────────────┘
```

**Orchestration** decides what happens when. **Execution** does bounded work. **State** persists everything to files.

---

## Components

### GSD Skills

| Skill | Purpose |
|-------|---------|
| **gsd-init** | Initialize projects via structured interview |
| **gsd-orchestrator** | Route commands, maintain state awareness |
| **gsd-phase-planner** | Two modes: discuss strategy, then create formal plan |
| **gsd-phase-executor** | Run Ralph Loop iterations within token budget |
| **gsd-phase-verifier** | Guide UAT, handle pass/fail |
| **gsd-state-manager** | Track decisions, trade-offs, open questions |

### Ralph Loop Pattern

Each iteration:
1. **Read** bounded context (~3,500 tokens)
2. **Execute** single task with completion promise
3. **Write** compressed handoff to disk

Context never accumulates. Quality never degrades.

### Seven Principles

1. **File-Based State Persistence** – State lives in files, not context
2. **Bounded Context Execution** – Hard 3,500 token limit per iteration
3. **Completion Promises** – Machine-verifiable progress markers
4. **Summarize, Don't Transcribe** – 5-10x compression on handoffs
5. **Human-in-the-Loop** – Quality gates at phase boundaries
6. **Two-Mode Planning** – Discuss before you plan
7. **Schema-Enforced Handoffs** – JSON schemas prevent drift

Remove any one and the others degrade. They interlock.

---

## Directory Structure

```
project/
├── PROJECT.md           # Vision, constraints, criteria
├── REQUIREMENTS.md      # Detailed requirements
├── ROADMAP.md           # Phase overview
├── ORCHESTRATION.md     # Current execution state
├── STATE.md             # Decisions, trade-offs
├── PROGRESS.txt         # Timestamped execution log
├── phases/
│   └── phase-n/
│       ├── PLAN.md      # Task breakdown
│       └── UAT.md       # Acceptance criteria
├── handoffs/
│   └── phase-n-handoff.json
├── research/            # Research outputs
├── docs/                # Documentation
└── templates/
    └── handoff-schemas/
```

---

## When to Use This

**Good fit:**
- Multi-phase projects (3+ phases)
- Workflows spanning multiple sessions
- Projects requiring auditability
- Teams without MLOps infrastructure
- Quality-critical applications

**Overkill for:**
- Single-turn questions
- Simple chatbot interactions
- Tasks completable in one pass

---

## Documentation

- [Usage Guide](docs/USAGE-GUIDE.md) – Complete walkthrough with examples
- [Skill Documentation](docs/skills/) – Individual skill READMEs
- [Error Recovery](docs/ERROR-RECOVERY.md) – When things go wrong
- [Quick Start](docs/QUICK-START.md) – Get running fast

---

## The Paper

This framework is documented in an academic paper:

**"Bounded Context Orchestration: A Framework for Multi-Phase LLM Workflows Without Context Exhaustion"**

The paper was written using the methodology it describes – six phases, fifty-plus bounded iterations, consistent quality throughout.

See `/paper/BOUNDED-CONTEXT-ORCHESTRATION.md`

---

## License

MIT License. Use it. Modify it. Build on it.

---

## Contributing

Issues and PRs welcome. The framework improves through use.

Key areas:
- Automatic schema generation
- Adaptive token budgets
- Parallel execution support
- Formal verification tools

---

*MoxyWolf LLC | Wild intelligence through structured recursion.*
