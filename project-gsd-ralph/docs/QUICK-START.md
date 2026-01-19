# Quick-Start Guide: GSD + Ralph Loop Framework

**Project:** GSD Ralph
**Phase:** 3 - Gap Analysis & Skill Development
**Task:** 3.5 - Quick-Start Guide
**Created:** 2026-01-19

---

## What is GSD + Ralph Loop?

**GSD (Get Shit Done)** is an orchestration framework for managing complex AI projects through structured phases.

**Ralph Loop** is an execution pattern that prevents context exhaustion by treating the LLM as a stateless function with bounded input (~3500 tokens per iteration).

**Together** they enable arbitrarily complex LLM workflows that maintain quality from start to finish.

---

## When to Use This Framework

### ✅ Use GSD + Ralph Loop When:

- Your workflow has **4+ distinct phases**
- You need **consistent quality** across all phases
- The project **spans multiple sessions**
- You need **auditable state** (debugging, compliance)
- **Human verification** is required at gates

### ❌ Don't Use When:

- Task completes in **single interaction**
- **Real-time response** required (<1 second)
- Simple **Q&A or chat** scenarios
- No need for **state persistence**

### Decision Tree:

```
Is your task multi-phase (3+ steps)?
├─ No ─► Standard prompting is fine
└─ Yes ─► Does quality matter in later phases?
           ├─ No ─► Standard chaining may work
           └─ Yes ─► Does state need to persist?
                     ├─ No ─► Prompt chaining with summarization
                     └─ Yes ─► Use GSD + Ralph Loop ✓
```

---

## Getting Started (5 Minutes)

### Step 1: Create Project Structure

```bash
mkdir my-project
cd my-project

# Create canonical directories
mkdir -p phases handoffs research todos outputs
```

### Step 2: Initialize Core Files

Create these files:

**PROJECT.md** - Your project's north star
```markdown
# Project: [Name]

## Vision
[What success looks like]

## Constraints
[Technical, timeline, budget limitations]

## Success Criteria
[How you'll know it's done]
```

**ROADMAP.md** - Phase structure
```markdown
# Roadmap

| Phase | Name | Objective | Status |
|-------|------|-----------|--------|
| 1 | [Name] | [What it achieves] | ⏳ Pending |
| 2 | [Name] | [What it achieves] | ⏳ Pending |
| ... | | | |
```

**ORCHESTRATION.md** - Current state
```markdown
# Orchestration State

current_phase: 1
status: planning
```

**PROGRESS.txt** - Execution log (append-only)
```
=== PROJECT INITIALIZED | YYYY-MM-DD ===
```

### Step 3: Create Your First Phase Plan

Create `phases/phase-1/PLAN.md`:

```markdown
# Phase 1 Plan: [Name]

## Tasks

### Task 1.1: [Name]
- **Objective:** [What to accomplish]
- **Completion Promise:** [MARKER_STRING]
- **Max Iterations:** 3

### Task 1.2: [Name]
- **Objective:** [What to accomplish]
- **Completion Promise:** [MARKER_STRING]
- **Max Iterations:** 3
```

---

## Your First Execution (Ralph Loop)

### The Pattern

Each iteration follows this cycle:

```
1. LOAD (bounded context ~3500 tokens)
   ├── Project summary (300 tokens)
   ├── Current task (500 tokens)
   ├── Last handoff (500 tokens)
   └── Skill instructions (1500 tokens)

2. EXECUTE (run the skill)

3. WRITE (compressed handoff ~500 tokens)
   ├── Status and completion promise
   ├── Summary of what was done
   ├── Key data for next phase
   └── Context hint for next iteration

4. CHECK (promise met?)
   ├── Yes → Next task
   └── No → Another iteration
```

### Example Iteration

**Loading context:**
```
PROJECT SUMMARY:
Building documentation pipeline with quality gates.

CURRENT TASK:
Task 1.1: Analyze source files
Completion Promise: ANALYSIS_COMPLETE

LAST HANDOFF:
(none - first task)

SKILL INSTRUCTIONS:
[Skill-specific guidance]
```

**After execution, write to PROGRESS.txt:**
```
=== ITERATION 1 | 2026-01-19T14:30:00Z ===
PHASE: 1 - Discovery
TASK: 1.1 - Analyze source files
STATUS: COMPLETE
COMPLETION_PROMISE: ANALYSIS_COMPLETE
PROMISE_MET: Yes
HANDOFF SUMMARY: Analyzed 12 source files. Key patterns identified: modular architecture, event-driven. Full analysis in research/analysis.md.
OUTPUT FILES: research/analysis.md
NEXT ACTION: Proceed to Task 1.2
===
```

---

## Common Patterns

### Pattern 1: Research → Synthesis → Output

```
Phase 1: Research
├── Task 1.1: Gather sources
├── Task 1.2: Analyze sources
└── Task 1.3: Compile findings

Phase 2: Synthesis
├── Task 2.1: Identify themes
├── Task 2.2: Build framework
└── Task 2.3: Validate logic

Phase 3: Output
├── Task 3.1: Draft document
├── Task 3.2: Review and refine
└── Task 3.3: Finalize
```

### Pattern 2: Problem Validation Pipeline

```
Phase 1: Problem Extraction
└── Handoff: problem_statement.json

Phase 2: Market Research
└── Handoff: research_findings.json

Phase 3: Report Generation
└── Output: validation_report.docx
```

### Pattern 3: Content Generation with Quality Gates

```
Phase 1: Planning
├── Human gate: Approve outline

Phase 2: Drafting
├── Iterative writing (section by section)

Phase 3: Review
├── Human gate: Approve draft

Phase 4: Polish
└── Final deliverable
```

---

## Handoff Best Practices

### DO:

```json
{
  "status": "COMPLETE",
  "summary": "Analyzed 6 skills. Token budget: 3500. Key finding: bounded context maintains quality.",
  "key_data": {
    "skills_count": 6,
    "token_budget": 3500
  },
  "output_file": "research/analysis.md",
  "next_hint": "Ready for theoretical synthesis"
}
```

### DON'T:

```json
{
  "full_analysis": "[10,000 tokens of raw output]",
  "all_search_results": "[...]",
  "intermediate_reasoning": "[...]"
}
```

### Compression Rules:

1. **Summarize, don't transcribe** – 10K output → 500 token summary
2. **Extract, don't explain** – Data, not methodology
3. **Pointers, not payloads** – File paths, not file contents
4. **Structure over prose** – JSON > paragraphs

---

## Troubleshooting

### "Quality degrades in later phases"

**Cause:** Context accumulating beyond bounds
**Fix:**
- Check you're reading only last handoff
- Compress handoffs more aggressively
- Verify token counts per component

### "Task never completes"

**Cause:** Completion promise not appearing
**Fix:**
- Check promise string matches exactly
- Verify skill can produce the promise
- Lower complexity or split task

### "Lost important context"

**Cause:** Over-compression in handoffs
**Fix:**
- Add missing data to structured JSON
- Use pointers to full outputs
- Adjust handoff schema

### "Session continuity broken"

**Cause:** State not properly persisted
**Fix:**
- Verify all files saved before ending
- Check ORCHESTRATION.md is current
- Review PROGRESS.txt for last state

---

## File Reference

| File | Purpose | Update Frequency |
|------|---------|------------------|
| PROJECT.md | Vision and constraints | Rarely |
| ROADMAP.md | Phase structure | Per phase |
| ORCHESTRATION.md | Current state | Every action |
| STATE.md | Decisions and trade-offs | As needed |
| PROGRESS.txt | Execution log | Every iteration |
| phases/phase-N/PLAN.md | Task breakdown | Once per phase |
| phases/phase-N/UAT.md | Verification criteria | Once per phase |
| handoffs/*.json | Inter-phase data | Per task |

---

## Next Steps

1. **Read the full research:** `research/GSD-RALPH-RESEARCH.md`
2. **Study the skill registry:** `research/SKILL-REGISTRY.md`
3. **Review error recovery:** `docs/ERROR-RECOVERY.md`
4. **Explore handoff schemas:** `templates/handoff-schemas/`

---

## Completion Status

**Task 3.5:** Quick-Start Guide - `QUICKSTART_COMPLETE`

**Sections:** 8
**Patterns:** 3
**Troubleshooting:** 4 common issues

