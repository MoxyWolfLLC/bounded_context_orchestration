# GSD + Ralph Loop Framework Research

**Project:** GSD Ralph
**Phase:** 1 - Discovery & Research
**Created:** 2026-01-19
**Status:** Complete

---

## Executive Summary

This research documents two complementary frameworks for building complex Claude systems that don't exhaust context: **Get Shit Done (GSD)** and the **Ralph Loop**.

**The Problem:** When chaining AI skills naively, context accumulates exponentially. A 4K-token first phase becomes 12K by phase two, 27K by phase three, and 39K+ by phase four. The model's attention diffuses. Quality degrades. Projects fail or produce poor results.

**The GSD Solution:** A 6-skill orchestration system that structures complex projects through interview-driven initialization, prerequisite-enforced routing, two-mode planning (discuss then plan), bounded context execution, human-in-the-loop verification, and explicit state persistence. Every significant state is written to disk, not held in conversation memory.

**The Ralph Loop Solution:** Treat the LLM as a stateless function. Each iteration reads only what it needs (~3500 tokens): project summary, current task, last handoff, and skill instructions. State persists via disk—PROGRESS.txt (append-only log), handoff JSON files (structured data), and iteration counters. Context stays bounded regardless of pipeline depth.

**Key Design Principles:**
1. **File-Based State Persistence** – All state externalized to disk
2. **Completion Promises** – Machine-verifiable success markers
3. **Compression Strategy** – Summarize, don't transcribe; extract, don't explain
4. **Human Checkpoints** – Verify, don't assume
5. **Bounded Context Execution** – ~3500 token ceiling per iteration

**Integration Observed:**
- GSD provides the meta-structure (phases, planning, verification)
- Ralph Loop provides the execution pattern (bounded iterations, handoffs)
- Academic Pipeline demonstrates domain-specific application
- Supporting skills (problem-statement-extractor, problem-awareness-discovery, moxywolf) chain together through compressed JSON handoffs

**Gaps Identified:**
- No automatic context compression when approaching limits
- Manual token counting required for handoffs
- Limited error recovery documentation
- No cross-session state resumption standard

**Conclusion:** The GSD + Ralph Loop combination solves a fundamental LLM limitation. By treating context as a scarce resource and externalizing state to disk, these frameworks enable arbitrarily complex projects that would otherwise fail from context exhaustion

---

## 1. GSD Framework Analysis

### 1.1 Overview

The Get Shit Done (GSD) framework is a comprehensive project orchestration system designed for complex, multi-phase AI workflows. It consists of 6 specialized skills that work together to manage projects from initialization through verification.

### 1.2 Core Skills Inventory

| Skill | Purpose | Primary Function |
|-------|---------|------------------|
| gsd-init | Project Bootstrap | Interactive interview → canonical file structure |
| gsd-orchestrator | Meta-routing | Command dispatch → skill invocation |
| gsd-phase-planner | Phase Design | Discussion + formal plan creation |
| gsd-phase-executor | Execution Engine | Ralph Loop implementation |
| gsd-phase-verifier | Quality Gate | UAT checklist verification |
| gsd-state-manager | Memory Persistence | Decisions, trade-offs, questions |

---

### 1.3 Skill-by-Skill Analysis

#### 1.3.1 gsd-init

**Architecture:**
- Single SKILL.md file (~574 lines)
- Template-driven document generation
- 8-phase execution flow

**Mechanics:**
- Interactive interview pattern (one question at a time)
- Creates 7 canonical files: PROJECT.md, REQUIREMENTS.md, ROADMAP.md, ORCHESTRATION.md, STATE.md, iteration.txt, PROGRESS.txt
- Creates 5 directories: phases/, handoffs/, research/, todos/, outputs/

**Execution Flow:**
1. Project Setup (get name, create directories)
2. Vision Interview (outcome, stakeholder, problem)
3. Constraints Interview (technical, timeline, budget, brand)
4. Success Criteria (done vs. success definitions)
5. Requirements (functional, non-functional, exclusions)
6. Phase Identification (suggest phases, allow modification)
7. Initial State Setup (create tracking files)
8. Summary (display what was created)

**Design Rationale:**
- One question at a time prevents overwhelm
- Templates ensure consistency across projects
- MoxyWolf integration as default brand option
- Validation checklist before completion

**Integration Points:**
- OUTPUT → All other GSD skills depend on files created here
- OUTPUT → ORCHESTRATION.md drives gsd-orchestrator routing
- OUTPUT → STATE.md used by gsd-state-manager
- OUTPUT → ROADMAP.md used by gsd-phase-planner

**Key Pattern: Interview-Driven Initialization**
The skill embodies a principle: *capture human intent before generating structure*. Rather than making assumptions, it systematically extracts vision, constraints, and success criteria.

---

#### 1.3.2 gsd-orchestrator

**Architecture:**
- Meta-orchestrator (router, not executor)
- Command-based dispatch table
- Inline execution for simple commands (status, advance)

**Mechanics:**
- Parses `/gsd [command]` syntax
- Routes to specialized skills based on command
- Maintains awareness of canonical file structure
- Natural language routing fallback

**Command Routing Table:**
| Command | Target Skill | Prerequisites |
|---------|-------------|---------------|
| `/gsd init` | gsd-init | None |
| `/gsd status` | (inline) | Project exists |
| `/gsd discuss-phase [N]` | gsd-phase-planner | Project initialized |
| `/gsd plan-phase [N]` | gsd-phase-planner | Discussion complete |
| `/gsd execute-phase [N]` | gsd-phase-executor | PLAN.md exists |
| `/gsd verify-phase [N]` | gsd-phase-verifier | SUMMARY.md exists |
| `/gsd state` | gsd-state-manager | Project initialized |
| `/gsd decide` | gsd-state-manager | Project initialized |

**Design Rationale:**
- Single entry point for all GSD operations
- Prerequisite checking prevents out-of-order execution
- After every command: update ORCHESTRATION.md, suggest next step
- "The orchestrator doesn't modify existing skills - it orchestrates them"

**Integration Points:**
- INPUT ← All `/gsd` commands from user
- OUTPUT → Routes to appropriate skill
- READ/WRITE → ORCHESTRATION.md (state tracking)
- READ → PROJECT.md, ROADMAP.md (context awareness)

**Key Pattern: Routing with Prerequisites**
The orchestrator enforces workflow discipline by checking prerequisites before routing. This prevents common errors like trying to execute before planning.

---

#### 1.3.3 gsd-phase-planner

**Architecture:**
- Two-mode skill (Discuss + Plan)
- ~552 lines with extensive templates
- Skill mapping logic for automation

**Mechanics - Discuss Mode:**
1. Load minimal context (~300 tokens)
2. Present phase objective from ROADMAP.md
3. Ask strategic questions (adapted by phase type)
4. Identify skills needed
5. Explore trade-offs if conflicting constraints
6. Document decisions to STATE.md
7. Transition prompt to Plan mode

**Mechanics - Plan Mode:**
1. Load context + discussion decisions
2. Verify discussion complete (fail if not)
3. Map skills to tasks
4. Create task breakdown (YAML-structured)
5. Map dependencies (Mermaid diagram)
6. Generate UAT criteria
7. Write PLAN.md + UAT.md
8. Update ORCHESTRATION.md

**Phase-Type Adaptive Questions:**
- Research/Discovery: depth, sources, platforms
- Content Creation: format, length, examples
- Development/Build: technical approach, speed vs quality
- Universal: constraints, autonomy level

**Skill Matching Logic:**
| Phase Contains | Suggested Skills |
|----------------|------------------|
| "problem", "validate" | problem-statement-extractor |
| "market", "research" | problem-awareness-discovery |
| "blog", "article" | voice-injection, moxywolf |
| "document", "report" | moxywolf, docx |
| "academic", "paper" | academic-pipeline-orchestrator |

**Design Rationale:**
- "Always run Discuss before Plan" - planning without discussion leads to wrong plans
- Task breakdown includes: id, name, skill, input_source, max_iterations, completion_promise, verification_criteria, failure_handling
- UAT criteria generated from phase objective + success criteria

**Integration Points:**
- INPUT ← ROADMAP.md (phase definitions)
- INPUT ← STATE.md (prior decisions)
- OUTPUT → STATE.md (discussion decisions)
- OUTPUT → phases/phase-N/PLAN.md
- OUTPUT → phases/phase-N/UAT.md
- OUTPUT → ORCHESTRATION.md (status updates)

**Key Pattern: Two-Mode Planning**
Separating discussion from planning ensures strategic alignment before committing to a formal plan. The discussion captures *why*, the plan captures *what* and *how*.

---

#### 1.3.4 gsd-phase-executor

**Architecture:**
- Ralph Loop implementation (core pattern)
- ~590 lines with detailed protocol
- Hard token bounds for context management

**Core Principle - Ralph Loop:**
```
Problem: LLM context windows fill up during complex work
Solution: Externalize state to disk

Each iteration reads only:
- Project summary (~300 tokens)
- Current task from PLAN (~500 tokens)
- Last handoff (~500 tokens)
- Skill instructions (~1500 tokens)
Total: ~3000-3500 tokens per iteration
```

**Mechanics - Main Execution Loop:**
1. **Context Loading** (bounded to ~3500 tokens)
2. **Skill Execution** (construct prompt with skill docs)
3. **Process Output** (check completion promise)
4. **Write Handoffs** (PROGRESS.txt + JSON)
5. **Completion Check** (promise met → next task)
6. **Blocking Check** (create BLOCKED.md if needed)
7. **Iteration Advance** (increment counter)
8. **Max Iterations Check** (handle non-completion)

**Hard Token Bounds:**
| Component | Max Tokens |
|-----------|------------|
| PROJECT_SUMMARY | 300 |
| PLAN_TASK | 500 |
| INPUT (handoff) | 500 |
| SKILL_DOC | 1500 |
| BUFFER | 700 |
| **TOTAL** | **3500** |

**Compression Strategy:**
1. Extract structured data first (JSON, key-value)
2. Keep critical prose (decisions, next actions, file refs)
3. Discard (historical context, intermediate reasoning)

**Handoff Format (PROGRESS.txt):**
```
=== ITERATION {N} | {timestamp} ===
PHASE: {N} - {phase_name}
TASK: {task.id} - {task.name}
SKILL: {task.skill}
STATUS: {COMPLETE|IN_PROGRESS|BLOCKED}
COMPLETION_PROMISE: {promise}
PROMISE_MET: {Yes|No}
HANDOFF SUMMARY: {compressed_output}
OUTPUT FILES: {file_list}
NEXT ACTION: {next_action}
===
```

**Error Recovery:**
- Context Overflow: Stop, compress, checkpoint, resume
- Skill Failure: Log, retry if sensible, BLOCKED.md if not
- File System Errors: Backup location, continue if possible

**Completion Promises by Skill:**
| Skill | Promise |
|-------|---------|
| problem-statement-extractor | `EXTRACTION_COMPLETE` |
| problem-awareness-discovery | `RESEARCH_COMPLETE` |
| moxywolf | `CONTENT_COMPLETE` |
| docx | `DOCUMENT_GENERATED` |

**Design Rationale:**
- "Skills are documentation, not code" - skill instructions become part of the prompt
- Append-only PROGRESS.txt ensures no data loss
- JSON handoffs enable structured recovery
- Max iterations prevent infinite loops

**Integration Points:**
- INPUT ← phases/phase-N/PLAN.md
- INPUT ← handoffs/phase-N/*.json (prior handoffs)
- INPUT ← Skill SKILL.md files
- OUTPUT → handoffs/phase-N/*.json
- OUTPUT → PROGRESS.txt (append)
- OUTPUT → phases/phase-N/SUMMARY.md
- OUTPUT → ORCHESTRATION.md

**Key Pattern: Bounded Context Execution**
This is the heart of the Ralph Loop. By externalizing state to disk and loading only what's needed for each iteration, the system can handle arbitrarily complex projects without context exhaustion.

---

#### 1.3.5 gsd-phase-verifier

**Architecture:**
- Interactive UAT workflow (~559 lines)
- Walk-through checklist pattern
- Failure handling with options

**Mechanics:**
1. Load context (SUMMARY.md + UAT.md)
2. Present execution summary
3. Walk through each UAT criterion
4. Collect results (pass/fail/skip)
5. Handle failures (fix, re-plan, accept, abort)
6. Write VERIFICATION.md if passed
7. Update ROADMAP.md + ORCHESTRATION.md

**User Response Options:**
| Response | Action |
|----------|--------|
| `yes` | Mark ✅, continue |
| `no` | Mark ❌, collect reason + severity |
| `skip` | Mark ⏭️, warning at end |

**Failure Severity Levels:**
1. Critical - must fix before proceeding
2. Major - should fix, could proceed with risk
3. Minor - nice to have, can proceed

**Failure Handling Options:**
1. Fix specific tasks and re-execute
2. Re-plan and re-execute entire phase
3. Accept and proceed anyway (documented)
4. Abort verification

**Criterion Types:**
- Deliverable Verification (files exist, have content)
- Quality Verification (brand voice, no placeholders)
- Functional Verification (links work, features function)
- Objective Verification (phase goals met)

**Design Rationale:**
- Interactive walk-through ensures human verification
- Severity classification enables informed decisions
- Accept-with-documentation allows pragmatic progress
- Rollback support enables clean recovery

**Integration Points:**
- INPUT ← phases/phase-N/SUMMARY.md
- INPUT ← phases/phase-N/UAT.md
- INPUT ← outputs/ (deliverables to verify)
- OUTPUT → phases/phase-N/VERIFICATION.md
- OUTPUT → ROADMAP.md (status update)
- OUTPUT → ORCHESTRATION.md (status update)
- OUTPUT → STATE.md (if failures accepted)

**Key Pattern: Human-in-the-Loop Verification**
The verifier ensures quality gates are human-controlled. It doesn't auto-pass; it requires explicit human confirmation for each criterion.

---

#### 1.3.6 gsd-state-manager

**Architecture:**
- Multi-mode state tracking (~649 lines)
- STATE.md as persistent memory
- Structured templates for each entry type

**Modes:**
| Command | Mode | Purpose |
|---------|------|---------|
| `/gsd state` | View | Display current state |
| `/gsd state --report [type]` | Report | Filtered view |
| `/gsd decide [topic]` | Decision | Document a choice |
| `/gsd question [topic]` | Question | Log open question |
| `/gsd tradeoff [what]` | Trade-off | Record accepted sacrifice |

**Decision Capture Flow:**
1. Context gathering (why needed now)
2. Options enumeration
3. Selection
4. Rationale
5. Impacts
6. Reversibility assessment
7. Write to STATE.md

**Question Tracking:**
- Context and possible answers
- Blocking status (with phase/task reference)
- Owner and deadline
- Resolution workflow

**Trade-off Documentation:**
- What was sacrificed
- What was gained
- Rationale
- Monitoring signals

**Auto-Extraction:**
During execution, can auto-extract decisions from handoffs containing decision markers.

**Design Rationale:**
- "STATE.md is the project's memory" - persists across sessions
- Projects fail when decisions get forgotten or contradicted
- Structured capture prevents ambiguity
- Blocking questions explicitly halt progress

**Integration Points:**
- READ/WRITE → STATE.md
- INTEGRATION → gsd-phase-planner (loads prior decisions)
- INTEGRATION → gsd-phase-executor (checks blocking questions)
- INTEGRATION → gsd-phase-verifier (references trade-offs)

**Key Pattern: Explicit State Persistence**
Rather than relying on conversation context, all significant state is explicitly captured in structured format. This enables session continuity and prevents decision amnesia.

---

### 1.4 GSD System Integration Map

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER COMMANDS                             │
│                    (/gsd init, /gsd status, etc.)               │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      gsd-orchestrator                            │
│              (Routes commands, checks prerequisites)             │
└────┬──────────┬──────────┬──────────┬──────────┬───────────────┘
     │          │          │          │          │
     ▼          ▼          ▼          ▼          ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐
│gsd-init │ │gsd-phase│ │gsd-phase│ │gsd-phase│ │gsd-state-   │
│         │ │-planner │ │-executor│ │-verifier│ │manager      │
└────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘
     │          │          │          │               │
     ▼          ▼          ▼          ▼               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CANONICAL FILES                             │
│  PROJECT.md | REQUIREMENTS.md | ROADMAP.md | ORCHESTRATION.md   │
│  STATE.md | PROGRESS.txt | iteration.txt                        │
│  phases/phase-N/PLAN.md | UAT.md | SUMMARY.md | VERIFICATION.md │
│  handoffs/phase-N/*.json                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 1.5 GSD Design Philosophy

**Core Principles Extracted:**

1. **File-Based State Persistence**
   - All significant state written to disk
   - Survives session boundaries
   - Enables recovery and debugging

2. **Structured Templates**
   - Consistent document formats
   - Predictable parsing
   - Clear expectations

3. **Human-in-the-Loop**
   - Interview-driven initialization
   - Explicit verification required
   - Decisions captured with rationale

4. **Prerequisite Enforcement**
   - Workflow discipline through checks
   - Prevents out-of-order execution
   - Clear error messages with guidance

5. **Bounded Context Execution**
   - Ralph Loop pattern
   - ~3500 token ceiling per iteration
   - Compression strategies for overflow

6. **Completion Promises**
   - Explicit success markers
   - Machine-verifiable completion
   - Prevent premature advancement

---

## 2. Ralph Loop Pattern Analysis

### 2.1 The Problem: Context Exhaustion

When chaining skills in a naive implementation, context accumulates exponentially:

```
Phase 1: 4K tokens → Output: 8K
Phase 2: 4K + 8K = 12K → Output: 15K
Phase 3: 4K + 8K + 15K = 27K → Output: 12K
Phase 4: 4K + 8K + 15K + 12K = 39K → Quality degrades
```

By phase 5-6, you're at 50-80K tokens. The model's attention diffuses, quality degrades, and the system either fails or produces poor results.

**Symptoms of Context Exhaustion:**
- Repeated content or circular reasoning
- Forgetting earlier instructions
- Inconsistent tone or approach
- Missing critical details from early phases
- Failure to complete tasks

### 2.2 The Solution: Ralph Loop

The Ralph Loop treats the LLM as a **stateless function** with **fresh, bounded context** each iteration. State persists via disk, not context accumulation.

```
Phase 1: 4K tokens → Write 500 token handoff
Phase 2: 4K + 500 = 4.5K → Write 500 token handoff
Phase 3: 4K + 500 + 800 JSON = 5.3K → Write 500 token handoff
Phase 4: 4K + 500 + 500 + 1K JSON = 6K → Complete
```

**Context stays bounded regardless of pipeline depth.**

### 2.3 Core Mechanism

#### 2.3.1 What Each Phase Reads

| Phase | Reads | Tokens |
|-------|-------|--------|
| 1 | User input | ~500 |
| 2 | Last handoff from PROGRESS.txt | ~500 |
| 3 | Last handoff + phase1-to-phase2.json | ~1300 |
| 4 | Last handoff + both JSON files | ~2000 |

Plus skill instructions (~1500) and system prompt (~2000). **Total: ~5-6K per phase.**

#### 2.3.2 What Each Phase Writes

**PROGRESS.txt entry** (~500 tokens):
- Status and completion promise
- Compressed summary of what was done
- Key data for next phase
- File path to full output
- Context hint for next phase

**Structured JSON** (when needed):
- Machine-readable data
- Key-value pairs next phase consumes
- No prose, no redundancy

### 2.4 Compression Strategy

#### 2.4.1 DO Include:
- Completion status
- Data the NEXT phase specifically needs
- File paths to full outputs
- Key extracted values
- One-sentence context hint

#### 2.4.2 DON'T Include:
- Full search results
- Raw API responses
- Methodology (that's in skill docs)
- Intermediate calculations
- Anything next phase won't read

#### 2.4.3 Token Budgets

| Component | Target | Max |
|-----------|--------|-----|
| Handoff summary | 400 | 500 |
| Context hint | 30 | 50 |
| Structured JSON | 500 | 1000 |
| Total per phase load | 5000 | 7000 |

### 2.5 Compression Examples

#### Summarize, Don't Transcribe

❌ **Full search results:**
```
Reddit thread 1: Title "Why is X so hard" - OP describes spending
20 hours on manual work, 47 upvotes, comments include...
[continues for 3000 tokens]
```

✅ **Compressed:**
```
Reddit findings (47 threads):
- Pain intensity: HIGH (20+ hrs/week cited)
- Top phrase: "Why is X so hard" (12 uses)
- Full analysis: outputs/market-research.md
```

#### Extract, Don't Explain

❌ **Methodology description:**
```
We used COMPOSIO_SEARCH_TRENDS to analyze volume over 12 months,
comparing problem-language against solution-language following
the awareness discovery methodology...
```

✅ **Just the data:**
```
Trends: Problem searches +34% YoY, Solution +12% YoY, ratio 2.8:1
```

#### Structured Over Prose

❌ **Scores in prose:**
```
The validation scored problem existence at 9 out of 10 based on
strong evidence. Frequency scored 8 because...
```

✅ **JSON:**
```json
{
  "problem_exists": {"score": 9, "evidence": "67 threads, 12 articles"},
  "problem_frequent": {"score": 8, "evidence": "Daily occurrence"}
}
```

### 2.6 Pointers Over Payloads

Instead of passing full outputs, pass:
- 500-token summary
- File path to full output
- Specific values next phase needs

If next phase needs full output, it reads the file directly. But most phases only need the handoff.

### 2.7 Ralph Loop in Practice: ralph-problem-validation-orchestrator

The `ralph-problem-validation-orchestrator` is a complete implementation demonstrating the pattern across 4 phases:

```
Phase 1: Initial Assessment (orchestrator) → ASSESSMENT_COMPLETE
Phase 2: Problem Extraction (problem-statement-extractor) → EXTRACTION_COMPLETE
Phase 3: Market Research (problem-awareness-discovery) → RESEARCH_COMPLETE
Phase 4: Report Generation (moxywolf + docx) → REPORT_COMPLETE
```

#### 2.7.1 Canonical Files

| File | Purpose |
|------|---------|
| `ORCHESTRATION.md` | Pipeline state, current phase |
| `PROGRESS.txt` | Append-only handoff log |
| `iteration.txt` | Current iteration counter |
| `handoffs/*.json` | Structured data between phases |

#### 2.7.2 Handoff Schemas

The orchestrator defines strict JSON schemas for inter-phase communication:

**phase1_to_phase2.json:**
```json
{
  "refined_input": "string (max 500 chars)",
  "suspected_icp": "string",
  "context": {
    "industry": "string",
    "geography": "string",
    "company_size": "string"
  }
}
```

**phase2_to_phase3.json:**
```json
{
  "problem_statement": {
    "problem": "string (max 200 chars)",
    "context": "string (max 200 chars)",
    "impact": "string (max 200 chars)",
    "evidence": "string (max 200 chars)",
    "success": "string (max 200 chars)"
  },
  "searchable_elements": {
    "core_pain": "string (max 100 chars)",
    "symptom": "string (max 100 chars)",
    "consequence": "string (max 100 chars)"
  }
}
```

**phase3_to_phase4.json:**
```json
{
  "validation_rating": {
    "rating": "WEAK|UNCERTAIN|PROMISING|VALIDATED|EXCEPTIONAL",
    "percentage": "number 0-100",
    "score": "number 0-60"
  },
  "icp_profile": {
    "primary_persona": "string",
    "job_titles": ["array", "max 5"],
    "pain_intensity": "LOW|MEDIUM|HIGH"
  },
  "key_findings": [
    {"finding": "string", "source": "string"}
  ],
  "language_gold": ["exact phrases from users"]
}
```

#### 2.7.3 Max Iterations Per Phase

| Phase | Max Iterations | Purpose |
|-------|----------------|---------|
| 1 | 3 | Initial assessment (simple) |
| 2 | 5 | Problem extraction (moderate) |
| 3 | 10 | Market research (complex, many queries) |
| 4 | 3 | Report generation (straightforward) |
| **Total** | **21** | Safety ceiling |

### 2.8 Ralph Loop vs. GSD Integration

The GSD framework's `gsd-phase-executor` implements the Ralph Loop pattern but generalizes it:

| Aspect | Ralph Loop (standalone) | GSD Phase Executor |
|--------|-------------------------|-------------------|
| Scope | Single pipeline (4 phases) | Any project (N phases) |
| Skills | Fixed (3 skills chained) | Dynamic (skill per task) |
| Handoff format | Custom JSON schemas | Generic YAML + JSON |
| Max iterations | Hardcoded per phase | Defined in PLAN.md |
| State tracking | ORCHESTRATION.md | ORCHESTRATION.md + STATE.md |

### 2.9 Design Principles

1. **Stateless Iteration**
   - Each iteration is self-contained
   - No reliance on conversation history
   - Fresh context, fresh attention

2. **Append-Only Logging**
   - PROGRESS.txt never modified, only appended
   - Creates audit trail
   - Enables debugging and recovery

3. **Schema-Enforced Handoffs**
   - JSON schemas define contracts
   - maxLength constraints enforce compression
   - Required fields ensure completeness

4. **Completion Promises**
   - Machine-verifiable success markers
   - Clear signal to advance
   - Prevents premature progression

5. **Pointers Over Payloads**
   - Pass file paths, not file contents
   - Defer full loading until needed
   - Keep handoffs minimal

### 2.10 Debugging Ralph Loop

**Quality degrades in later phases:**
- Likely reading full history instead of last handoff
- Fix: Read only last PROGRESS.txt entry

**Phase misses critical context:**
- Handoff too compressed
- Fix: Add missing data to structured JSON

**Pipeline runs forever:**
- Completion promise not appearing in output
- Fix: Check promise string matches exactly

**Redundant work:**
- Handoffs not clear about what was done
- Fix: Better "completed" markers and context hints

---

## 3. Academic Pipeline Analysis

### 3.1 Overview

The Academic Pipeline is an 8-stage orchestrated workflow for producing publication-ready academic content from bibliography input. It demonstrates sophisticated skill chaining with a key innovation: **voice and formatting are established BEFORE writing begins**, not retrofitted after.

### 3.2 Skills Inventory

| Stage | Skill | Purpose | Interactive? |
|-------|-------|---------|--------------|
| 1 | bibtex-theme-analyzer | Map research landscape → Mermaid diagram | No |
| 2 | academic-perspective-builder | Define writing strategy via interview | Yes (3 questions) |
| 3 | voice-injection | Capture author's authentic voice | Yes (multiple) |
| 4 | academia-formatting | Load formatting requirements | No |
| 5 | research-analyst | Structure plan with voice integration | Yes (approval) |
| 6 | research-writer | Iterative section writing | No (loops) |
| 7 | bibliography-generator | Format citations | No |
| 8 | professor | Critique and improvement plan | No |

### 3.3 Pipeline Flow

```
User Input (BibTeX)
        ↓
[Stage 1] bibtex-theme-analyzer → Mermaid diagram + TARGET_TITLE
        ↓
[Stage 2] academic-perspective-builder → perspective.json
        ↓
[Stage 3] voice-injection → voice_context.json (BEFORE writing)
        ↓
[Stage 4] academia-formatting → formatting_requirements.json (BEFORE writing)
        ↓
[Stage 5] research-analyst → handoff_for_writer.json (informed by voice + format)
        ↓
[Stage 6] research-writer (ITERATIVE) → accumulated_content
        ↓
[Stage 7] bibliography-generator → complete_document.md
        ↓
[Stage 8] professor → critique_report.md + improvement_plan.md
        ↓
User Output (Formatted document + Critique + Improvement roadmap)
```

### 3.4 Skill-by-Skill Analysis

#### 3.4.1 bibtex-theme-analyzer (Stage 1)

**Purpose:** Analyze bibliography and generate thematic visualization.

**Input:** BibTeX content (pasted or uploaded)

**Process:**
1. Retrieve BibTeX content (paginated if needed)
2. Analyze abstracts for recurring themes
3. Suggest TARGET_TITLE (2-3 options with rationale)
4. Extract and group themes hierarchically
5. Generate Mermaid diagram

**Output:**
- `theme_analysis.json` (metadata)
- `mermaid_diagram.md` (visualization)
- `detected_filename` (confirmed title)

**Design Rationale:** Starting with theme analysis provides the conceptual scaffold for all subsequent stages. The Mermaid diagram becomes a reference throughout the pipeline.

---

#### 3.4.2 academic-perspective-builder (Stage 2)

**Purpose:** Define writing strategy through interactive interview.

**Process (3-step interview):**

1. **Establish Writing Perspective**
   - Suggest 2-3 perspectives aligned with research
   - Options: Critical, Historical, Practical, Theoretical, Innovation-focused, Comparative
   - Reconfirm choice before proceeding

2. **Identify Target Market**
   - Academic researchers, industry professionals, students, policymakers
   - Specific background and expertise level
   - Reconfirm before proceeding

3. **Define Sub-themes**
   - Reference Mermaid diagram branches
   - Select 3-5 focused areas
   - Reconfirm all choices

**Output:** `perspective.json`
```json
{
  "topic": "...",
  "target_market": "...",
  "writing_perspective": "...",
  "sub_themes": ["...", "...", "..."]
}
```

**Design Rationale:** Human-in-the-loop strategy definition ensures content aligns with author's intent. All three components required - no skipping.

---

#### 3.4.3 voice-injection (Stage 3) - BEFORE Writing

**Purpose:** Capture author's authentic voice before any content generation.

**Interview Gathers:**
- Author's perspective on the topic
- Relevant lived experience (e.g., 22 years compliance, STIGViewer)
- Strong opinions/convictions about the research area
- Brand-specific angles and frameworks (MoxyWolf)
- Personal anecdotes or examples
- What the author would emphasize that generic AI wouldn't

**Output:** `voice_context.json`

**Design Rationale:** This is the key innovation. By capturing voice BEFORE writing, content is authentic from the start, not "fixed" afterward. This prevents the common AI problem of generic, voiceless prose.

---

#### 3.4.4 academia-formatting (Stage 4) - BEFORE Writing

**Purpose:** Establish formatting standards before writing begins.

**Defines:**
- Section structure for Academia.edu
- Citation style specifics
- Scholarly register expectations
- Voice preservation guidelines
- Anti-AI writing pattern requirements

**Output:** `formatting_requirements.json`

**Design Rationale:** Format constraints are inputs to writing, not post-processing. The writer applies them from the start.

---

#### 3.4.5 research-analyst (Stage 5)

**Purpose:** Create document structure informed by voice and format requirements.

**Inputs:**
- `perspective.json` (Stage 2)
- `voice_context.json` (Stage 3)
- `formatting_requirements.json` (Stage 4)
- BibTeX source
- Mermaid diagram

**Process:**
1. Load all context
2. Analyze research landscape (1000-2000 words)
3. Create structure plan aligned with perspective, market, and voice
4. Select 3-8 sources per section
5. Map voice integration points
6. Define citation rules

**Output:** `handoff_for_writer.json`
```json
{
  "section_title": "...",
  "structure_plan": ["Section 1", "Section 2", ...],
  "key_sources_planned": ["bibkey1", "bibkey2", ...],
  "voice_integration": {
    "dorian_perspectives": ["Section 1: experience X", ...],
    "moxywolf_frameworks": ["framework Y", ...],
    "writing_patterns": ["avoid phrase Z", ...]
  },
  "formatting_requirements": {...},
  "section_source_mapping": {...}
}
```

**User Review Point:** Structure shown with voice integration points. User approves or modifies.

---

#### 3.4.6 research-writer (Stage 6) - ITERATIVE

**Purpose:** Write content one section at a time with voice and format baked in.

**Iteration Pattern:**
```
WHILE current_section_index < total_sections:
  - Write current section (400-800 words)
  - Include 2-5 citations per rules
  - Apply voice integration points
  - Follow formatting requirements
  - Verify citations
  - Return JSON with section_content
  - Orchestrator appends to accumulated_content
  - Increment current_section_index
  IF is_complete: BREAK
```

**State Management:**
```json
{
  "current_section_index": 0,
  "sections_completed_count": 0,
  "sections_completed_titles": [],
  "accumulated_content": "",
  "all_citations_used": [],
  "iteration_count": 1
}
```

**Critical Rules:**
- NEVER write multiple sections in one iteration
- NEVER write actual bibliography (that's Stage 7)
- ALWAYS use real newlines, not `\\n`
- ALWAYS verify BibTeX data before citing

---

#### 3.4.7 bibliography-generator (Stage 7)

**Purpose:** Generate properly formatted bibliography.

**Process:**
1. Collect unique citations from all_citations_used
2. Retrieve full BibTeX entries
3. Format per citation_style (APA, Chicago, MLA)
4. Handle special cases (no author, multiple authors, web sources)
5. Alphabetize and integrate with document

**Output:** `complete_document.md` with formatted bibliography

---

#### 3.4.8 professor (Stage 8)

**Purpose:** Comprehensive 10-phase quality review.

**Critique Phases:**
1. AI Detection Analysis (should pass with authentic voice)
2. Document Integrity Check
3. Citation Format Analysis
4. Reference Completeness Audit
5. Logic & Argumentation Analysis
6. Methodological Rigor Assessment
7. Literature Review Critique
8. Evidence & Results Analysis
9. Writing Quality Assessment
10. Contribution Assessment

**Output:**
- `critique_report.md` (detailed scores)
- `improvement_plan.md` (actionable roadmap)

### 3.5 State Management

The Academic Pipeline maintains comprehensive state:

```json
{
  "pipeline_id": "uuid",
  "current_stage": 1-8,
  "status": "in_progress|complete|error|paused",
  "stage_1": { "complete": true, "output_file": "..." },
  "stage_2": { "complete": true, "perspective": {...} },
  "stage_3": { "complete": true, "voice_context": {...} },
  "stage_4": { "complete": true, "formatting_requirements": {...} },
  "stage_5": { "complete": true, "sections_planned": 6 },
  "stage_6": { "complete": false, "current_iteration": 3 },
  "stage_7": { "complete": false },
  "stage_8": { "complete": false }
}
```

Saved to `.pipeline_state.json` for resumption.

### 3.6 Key Innovation: Voice-First Writing

The Academic Pipeline's key differentiator from other content generation systems:

| Traditional Approach | Academic Pipeline Approach |
|---------------------|---------------------------|
| Generate generic content | Capture voice BEFORE writing |
| Retrofit voice/style | Bake voice into structure plan |
| Post-process formatting | Apply format during writing |
| Fix AI-isms after | Prevent AI-isms from the start |

**Result:** Content that reads as human-authored because it was informed by human voice from the beginning.

### 3.7 Comparison to Ralph Loop

| Aspect | Academic Pipeline | Ralph Loop (GSD) |
|--------|-------------------|------------------|
| Stages | 8 fixed stages | N configurable tasks |
| State Format | JSON state object | PROGRESS.txt + JSON handoffs |
| Iteration | Stage 6 only | Every task can iterate |
| Human Control | Interactive stages 2, 3, 5 | Check-ins per plan |
| Output | Single document + critique | Any deliverables |

**Key Difference:** Academic Pipeline is domain-specific (academic content). GSD + Ralph Loop is general-purpose.

### 3.8 Design Principles

1. **Voice Before Content**
   - Capture author perspective before any generation
   - Voice informs structure, not just style

2. **Format as Input**
   - Requirements defined upfront
   - No post-processing retrofitting

3. **Iterative Controlled Writing**
   - One section at a time
   - State tracked between iterations
   - Token safety through bounded sections

4. **Human Checkpoints**
   - Interactive stages for strategy
   - Approval gates before execution
   - Can pause and resume

5. **Quality Assurance Built-In**
   - Professor critique as final stage
   - AI detection should pass
   - Actionable improvement roadmap

---

## 4. Supporting Skills Analysis

The GSD + Ralph Loop framework chains together specialized skills. These three supporting skills demonstrate how domain-specific functionality integrates with the orchestration pattern.

### 4.1 problem-statement-extractor

**Purpose:** Extract and document rigorous, evidence-based problem statements using multi-perspective swarm analysis.

#### 4.1.1 Core Innovation: Multi-Persona Swarm Approach

The skill uses four cognitive personas to challenge and refine problem definitions:

| Persona | Role | Lens |
|---------|------|------|
| The Clarifier (Feynman) | Strip jargon, ensure clarity | "Can I explain this to a 12-year-old?" |
| The Evidence Seeker (Popper) | Validate claims with data | "What could prove this wrong?" |
| The Impact Analyst (Munger) | Articulate consequences | "Who bears the cost?" |
| The Scope Setter (Buffett) | Define boundaries | "Is this actually solvable?" |

**Why This Matters:** Each persona attacks the problem from a different angle. The Clarifier prevents jargon-hiding. The Evidence Seeker demands proof. The Impact Analyst ensures consequences are understood. The Scope Setter prevents scope creep.

#### 4.1.2 Execution Phases

```
Phase 1: Evidence Extraction (2-3 min)
  - Read entire input
  - Extract pain points, gaps, inefficiencies, failures
  - Identify affected parties
  - Capture success criteria

Phase 2: Multi-Perspective Analysis (5-8 min)
  - Run all four persona lenses
  - Each produces refined problem statement
  - Challenges cascade through perspectives

Phase 3: Synthesis & Validation (3-5 min)
  - Combine insights from all personas
  - Validate against 9-point checklist
  - Format using chosen structure

Phase 4: Anti-Solution Check (2 min)
  - Remove all solution language
  - Verify problem/solution separation
  - Ensure purely descriptive current state
```

#### 4.1.3 Output Formats

**Five-Sentence Format:**
```
Problem: [One sentence - core issue]
Context: [One sentence - who/when/where]
Impact: [One sentence - consequences]
Evidence: [One sentence - analysis/data]
Success: [One sentence - what "solved" means]
```

**Validation Checklist:**
- [ ] Problem clearly stated (not solution in disguise)
- [ ] Affected parties identified
- [ ] Context/trigger specified
- [ ] Impact/consequences articulated
- [ ] Evidence cited from analysis
- [ ] Success criteria defined
- [ ] Actionable
- [ ] Specific
- [ ] Separates problem from solution

#### 4.1.4 Integration with Ralph Loop

The problem-statement-extractor is Phase 2 of the ralph-problem-validation-orchestrator pipeline:

```
Phase 1 (orchestrator) → ASSESSMENT_COMPLETE
Phase 2 (problem-statement-extractor) → EXTRACTION_COMPLETE
Phase 3 (problem-awareness-discovery) → RESEARCH_COMPLETE
Phase 4 (moxywolf + docx) → REPORT_COMPLETE
```

**Handoff Schema (phase1-to-phase2.json):**
```json
{
  "refined_input": "string (max 500 chars)",
  "suspected_icp": "string",
  "context": {
    "industry": "string",
    "geography": "string"
  }
}
```

**Output Schema (phase2-to-phase3.json):**
```json
{
  "problem_statement": {
    "problem": "string (max 200 chars)",
    "context": "string (max 200 chars)",
    "impact": "string (max 200 chars)",
    "evidence": "string (max 200 chars)",
    "success": "string (max 200 chars)"
  },
  "searchable_elements": {
    "core_pain": "string (max 100 chars)",
    "symptom": "string (max 100 chars)",
    "consequence": "string (max 100 chars)"
  },
  "user_confirmed": true
}
```

**Key Constraint:** Max 200 characters per field enforces compression.

---

### 4.2 problem-awareness-discovery

**Purpose:** Validate problem hypothesis by discovering search signals, discussion patterns, and ICP characteristics across multiple data sources.

#### 4.2.1 Multi-Source Discovery Architecture

| Source | Tool | Purpose |
|--------|------|---------|
| Web | COMPOSIO_SEARCH_WEB | Content landscape, competitor positioning |
| Trends | COMPOSIO_SEARCH_TRENDS | Problem vs solution volume, rising queries |
| News | COMPOSIO_SEARCH_NEWS | Forcing functions, media framing |
| YouTube | SERPAPI_YOU_TUBE_SEARCH | Content gaps, creator landscape |
| Reddit | REDDIT_SEARCH_ACROSS_SUBREDDITS | Real language, ICP signals, emotional tone |

**Why Multiple Sources:** Single-source validation is unreliable. Triangulation across trends, discussions, and content provides robust signal.

#### 4.2.2 Workflow Phases

```
Phase 1: Problem Decomposition
  - Extract: CORE PAIN, SYMPTOM, CONSEQUENCE, CONTEXT, SUSPECTED SUFFERER

Phase 2: Query Generation
  - Problem-language (Stage 2): "why is [X] so [negative]"
  - Solution-language (Stage 3-4): "best [category] tools"
  - Generate 10-15 queries per awareness stage

Phase 3: Multi-Source Discovery
  - Execute via RUBE_MULTI_EXECUTE_TOOL
  - Collect signals from all sources

Phase 4: ICP Synthesis
  - Extract job titles from discussions
  - Identify company signals
  - Assess pain intensity
  - Gauge budget signals
  - Determine sophistication level

Phase 5: Validation Assessment
  - Score: Search signals exist?
  - Score: Discussion volume?
  - Score: Language consistency?
  - Score: Solution awareness level?
  - Score: ICP clarity?
  - Overall: Validated / Promising / Uncertain / Weak

Phase 6: Recommendations
  - Validated → Proceed to PR/FAQ
  - Promising → Narrow focus, 5-10 interviews
  - Uncertain → Reframe, primary research
  - Weak → Consider pivot

Phase 7: Report Generation
  - Comprehensive markdown report
  - Save to /mnt/user-data/outputs/
```

#### 4.2.3 Awareness Stage Mapping

The skill uses Eugene Schwartz's awareness model:

| Stage | Description | Search Behavior |
|-------|-------------|-----------------|
| 1 (Unaware) | Don't know problem exists | Cannot measure via search |
| 2 (Problem-Aware) | Know problem, not solutions | "Why is X so hard" |
| 3 (Solution-Aware) | Know solutions exist | "Best tools for X" |
| 4 (Product-Aware) | Know specific products | "[Product] vs [Product]" |
| 5 (Most Aware) | Know your product | "[Your Brand] pricing" |

**Critical Insight:** Low search volume doesn't mean bad idea. Some problems are experienced but not searched for. The skill includes guidance for interpreting weak signals.

#### 4.2.4 ICP Synthesis from Discussions

The skill extracts ICP characteristics from natural language:

```
Job titles: "As a [title]..."
Company signals: "We're a [size] [type]..."
Pain intensity: "losing sleep over...", "boss breathing down my neck..."
Budget signals: "can't afford...", "worth paying for..."
Sophistication: jargon usage, alternatives mentioned
```

**Output: Language Gold** - Exact phrases from users become copywriting material.

#### 4.2.5 Integration with Ralph Loop

**Receives from Phase 2:**
- Validated problem statement
- Searchable elements (core_pain, symptom, consequence)
- User-confirmed scope

**Passes to Phase 4:**
```json
{
  "validation_rating": {
    "rating": "WEAK|UNCERTAIN|PROMISING|VALIDATED|EXCEPTIONAL",
    "percentage": "0-100",
    "score": "0-60"
  },
  "icp_profile": {
    "primary_persona": "string",
    "job_titles": ["array"],
    "pain_intensity": "LOW|MEDIUM|HIGH"
  },
  "key_findings": [
    {"finding": "string", "source": "string"}
  ],
  "language_gold": ["exact phrases"]
}
```

---

### 4.3 moxywolf

**Purpose:** Apply MoxyWolf LLC brand guidelines for writing, visual creation, and document styling. The skill embodies "wild intelligence through structured recursion."

#### 4.3.1 Core Innovation: Human-Authentic Writing Guidelines

The skill's most significant contribution is its **anti-AI detection system** - a comprehensive set of rules to make AI-generated content read as human-authored.

**Why This Matters:** AI-generated content has identifiable patterns. The moxywolf skill explicitly defines and prohibits these patterns.

#### 4.3.2 Anti-Detection System

**Forbidden Phrases (AI Tells):**
- "It's worth noting that..."
- "It's important to understand..."
- "In today's rapidly changing..."
- "At its core..."
- "This raises important questions about..."
- "Moving forward..."
- "In order to..." (just use "to")

**Forbidden Transitions:**
| AI Transition | Human Alternative |
|---------------|-------------------|
| Furthermore | And, Plus, Also |
| Additionally | And, or delete |
| Moreover | What's more, or delete |
| In conclusion | Just conclude |
| Consequently | So, Which means |
| Nevertheless | But, Still, Yet |

**Mandatory Patterns:**
- Use contractions 80%+ of the time
- Vary sentence length aggressively (4-word to 40-word)
- Use fragments. Deliberately. For emphasis.
- Start sentences with conjunctions (And, But, Or)
- Break the "topic sentence" pattern
- Use asymmetric structure (not always threes)
- Replace abstract claims with specific examples

#### 4.3.3 Voice Principles

| Principle | Description | Example |
|-----------|-------------|---------|
| Anchored Boldness | Make declarative statements | "This approach transforms..." not "might potentially help..." |
| Recursive Precision | Layer ideas, parallel structure | "We trace patterns. We tune systems. We anchor truth." |
| Creative Restraint | Bold but controlled | "We design from recursion" not "We're wildly innovative disruptors!" |
| Spiritual Pragmatism | Depth with practicality | Balance philosophy with action |

#### 4.3.4 Document Styling (docx-js)

The skill provides complete JavaScript code for generating branded Word documents:

**Color System:**
```javascript
const CHARCOAL = "1C1C1C";     // Headings, table headers
const FIRE_ORANGE = "F77028";  // Subheadings, accents
const GRAY = "5D5D5D";         // Secondary text
const LIGHT_GRAY = "CCCCCC";   // Table borders
const WHITE = "FFFFFF";        // Table header text
```

**Helper Functions Provided:**
- `h1(text)` - Primary headings in Charcoal
- `h2(text)` - Secondary headings in Fire Orange
- `para(text)` - Standard paragraphs
- `boldPara(bold, normal)` - Mixed formatting
- `createTable(headers, rows, widths)` - Branded tables
- `calloutBox(lines)` - Orange left border emphasis

**Critical Rules:**
- Always use `ShadingType.CLEAR` (never SOLID)
- Set both `columnWidths` on table AND `width` on each cell
- Wrap `PageBreak` in `Paragraph`
- Use percentage width for tables

#### 4.3.5 Integration with Ralph Loop

The moxywolf skill is Phase 4 of the problem validation pipeline:

```
Phase 4: Report Generation
  - Read both handoff JSON files (phase1-to-phase2, phase2-to-phase3)
  - Generate Word document via docx-js
  - Apply MoxyWolf styling
  - Sections: Executive Summary, Problem Statement, Findings, Scorecard, Recommendations
  - Copy to /mnt/user-data/outputs/
  - Deliver via present_files
```

**Why Last:** Branding and formatting are applied after content is complete. Voice guidelines inform writing throughout, but final document styling happens at delivery.

#### 4.3.6 Rewrite Test

Before delivery, content is verified against:

1. Could this have been written by a careful human in one pass?
2. Are there three or more sentences of similar length in a row?
3. Is there a "topic sentence + supporting details" structure?
4. Are there any AI-tell phrases or transitions?
5. Would a reader in the target audience actually talk this way?

---

### 4.4 Supporting Skills Integration Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    RALPH LOOP ORCHESTRATION                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
    ┌────────────────────────┼────────────────────────────────┐
    │                        │                                │
    ▼                        ▼                                ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ problem-     │───▶│ problem-         │───▶│ moxywolf         │
│ statement-   │    │ awareness-       │    │                  │
│ extractor    │    │ discovery        │    │ (+ docx-js)      │
└──────────────┘    └──────────────────┘    └──────────────────┘
      │                      │                      │
      ▼                      ▼                      ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Validated    │    │ Market research  │    │ Branded Word     │
│ problem      │    │ ICP profile      │    │ document         │
│ statement    │    │ Awareness dist.  │    │ Final delivery   │
└──────────────┘    └──────────────────┘    └──────────────────┘
```

### 4.5 Design Patterns Across Supporting Skills

#### 4.5.1 Common Patterns

1. **Structured Output Formats**
   - All three skills produce well-defined output schemas
   - JSON for machine consumption, Markdown for human reading
   - Character limits enforce compression

2. **Validation Checklists**
   - problem-statement-extractor: 9-point validation
   - problem-awareness-discovery: 6-dimension scoring
   - moxywolf: 10-point authenticity checklist

3. **Human-in-the-Loop**
   - problem-statement-extractor: User confirms final statement
   - problem-awareness-discovery: Recommendations require human decision
   - moxywolf: Rewrite test for authenticity

4. **Multi-Source/Multi-Perspective**
   - problem-statement-extractor: 4 personas
   - problem-awareness-discovery: 5 data sources
   - moxywolf: Multiple voice principles

#### 4.5.2 Ralph Loop Compatibility

All three skills are designed for bounded context execution:

| Skill | Input Tokens | Output Tokens | Handoff Size |
|-------|--------------|---------------|--------------|
| problem-statement-extractor | ~500 (refined input) | ~1000 | ~800 (JSON) |
| problem-awareness-discovery | ~800 (handoff + searches) | ~2000 | ~1200 (JSON) |
| moxywolf | ~1200 (both handoffs) | Full document | N/A (terminal) |

**Key Observation:** Each skill reads compressed handoff, not full prior output. This is the Ralph Loop pattern in action.

---

### 4.6 Completion Status

**Task 1.4:** Supporting Skills Analysis - `SUPPORTING_SKILLS_ANALYZED`

---

## 5. Integration Map

### 5.1 Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERFACE                                  │
│                         (Commands, Conversations)                            │
└─────────────────────────────────────┬───────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            GSD ORCHESTRATION LAYER                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │ gsd-init    │ │ gsd-phase-  │ │ gsd-phase-  │ │ gsd-phase-  │           │
│  │             │ │ planner     │ │ executor    │ │ verifier    │           │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘ └──────┬──────┘           │
│         │               │               │               │                   │
│         └───────────────┴───────┬───────┴───────────────┘                   │
│                                 │                                           │
│                    ┌────────────┴────────────┐                              │
│                    │    gsd-orchestrator     │                              │
│                    │   (Command Routing)     │                              │
│                    └────────────┬────────────┘                              │
│                                 │                                           │
│                    ┌────────────┴────────────┐                              │
│                    │   gsd-state-manager     │                              │
│                    │  (Memory Persistence)   │                              │
│                    └─────────────────────────┘                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            RALPH LOOP EXECUTION                              │
│                                                                              │
│   Iteration 1        Iteration 2        Iteration 3        Iteration N      │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐       ┌──────────┐     │
│  │ Load     │       │ Load     │       │ Load     │       │ Load     │     │
│  │ Context  │──────▶│ Context  │──────▶│ Context  │──────▶│ Context  │     │
│  │ (~3.5K)  │       │ (~3.5K)  │       │ (~3.5K)  │       │ (~3.5K)  │     │
│  └────┬─────┘       └────┬─────┘       └────┬─────┘       └────┬─────┘     │
│       │                  │                  │                  │            │
│       ▼                  ▼                  ▼                  ▼            │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐       ┌──────────┐     │
│  │ Execute  │       │ Execute  │       │ Execute  │       │ Execute  │     │
│  │ Skill    │       │ Skill    │       │ Skill    │       │ Skill    │     │
│  └────┬─────┘       └────┬─────┘       └────┬─────┘       └────┬─────┘     │
│       │                  │                  │                  │            │
│       ▼                  ▼                  ▼                  ▼            │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐       ┌──────────┐     │
│  │ Write    │       │ Write    │       │ Write    │       │ Write    │     │
│  │ Handoff  │       │ Handoff  │       │ Handoff  │       │ Handoff  │     │
│  │ (~500)   │       │ (~500)   │       │ (~500)   │       │ (~500)   │     │
│  └──────────┘       └──────────┘       └──────────┘       └──────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            DOMAIN SKILL LAYER                                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Problem Validation Pipeline                                          │    │
│  │ ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐  │    │
│  │ │ problem-   │──▶│ problem-   │──▶│ moxywolf   │──▶│ DELIVERABLE│  │    │
│  │ │ statement- │   │ awareness- │   │ + docx     │   │            │  │    │
│  │ │ extractor  │   │ discovery  │   │            │   │            │  │    │
│  │ └────────────┘   └────────────┘   └────────────┘   └────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Academic Pipeline                                                    │    │
│  │ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    │    │
│  │ │Theme │▶│Persp-│▶│Voice │▶│Format│▶│Anal- │▶│Write │▶│Prof- │    │    │
│  │ │Anal. │ │ective│ │Inject│ │      │ │yst   │ │er    │ │essor │    │    │
│  │ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Other Domain Pipelines (Extensible)                                  │    │
│  │  • Content marketing pipeline                                        │    │
│  │  • Competitive analysis pipeline                                     │    │
│  │  • Technical documentation pipeline                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FILE SYSTEM STATE LAYER                            │
│                                                                              │
│  Canonical Files:                   Handoff Files:                          │
│  ├── PROJECT.md                     ├── handoffs/                           │
│  ├── REQUIREMENTS.md                │   ├── phase1-to-phase2.json           │
│  ├── ROADMAP.md                     │   ├── phase2-to-phase3.json           │
│  ├── ORCHESTRATION.md               │   └── ...                             │
│  ├── STATE.md                       │                                       │
│  ├── PROGRESS.txt (append-only)     Phase Files:                            │
│  └── iteration.txt                  ├── phases/phase-N/                     │
│                                     │   ├── PLAN.md                         │
│                                     │   ├── UAT.md                          │
│                                     │   ├── SUMMARY.md                      │
│                                     │   └── VERIFICATION.md                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Data Flow Through System

```
USER INPUT
    │
    ▼
┌───────────────────┐
│ gsd-init          │ ←── Creates PROJECT.md, ROADMAP.md, STATE.md
│ (Interview)       │     Sets up phase structure
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ gsd-phase-planner │ ←── Creates PLAN.md, UAT.md
│ (Discuss + Plan)  │     Maps skills to tasks
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐     ┌─────────────────────────────┐
│ gsd-phase-executor│────▶│ RALPH LOOP                  │
│ (Bounded Context) │     │                             │
└─────────┬─────────┘     │ Read: ~3500 tokens          │
          │               │  • Project summary (~300)   │
          │               │  • Current task (~500)      │
          │               │  • Last handoff (~500)      │
          │               │  • Skill docs (~1500)       │
          │               │                             │
          │               │ Execute: Domain Skill       │
          │               │                             │
          │               │ Write: ~500 token handoff   │
          │               │  • Status + promise         │
          │               │  • Compressed summary       │
          │               │  • File paths               │
          │               │  • Context hint             │
          │               └──────────────┬──────────────┘
          │                              │
          │◀─────────────────────────────┘
          │
          ▼
┌───────────────────┐
│ gsd-phase-verifier│ ←── Walks through UAT.md
│ (Human Approval)  │     Creates VERIFICATION.md
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ gsd-state-manager │ ←── Updates STATE.md
│ (Memory)          │     Tracks decisions, trade-offs
└───────────────────┘
```

### 5.3 Token Flow Analysis

| Component | Input | Processing | Output | Net Token Cost |
|-----------|-------|------------|--------|----------------|
| gsd-init | User responses | Interview | 7 files created | ~2000 |
| gsd-phase-planner | ROADMAP.md, STATE.md | Two-mode | PLAN.md, UAT.md | ~1500 |
| gsd-phase-executor | Last handoff (~500) | Ralph Loop | Handoff (~500) | **~3500/iteration** |
| gsd-phase-verifier | SUMMARY.md, UAT.md | Checklist | VERIFICATION.md | ~1000 |
| gsd-state-manager | Current state | Update | STATE.md | ~500 |

**Critical Insight:** The Ralph Loop bounds per-iteration cost to ~3500 tokens regardless of project complexity. A 10-phase project with 5 iterations each = 50 iterations, but each iteration still costs only ~3500 tokens.

### 5.4 Skill Chaining Patterns

#### Pattern 1: Sequential Pipeline (Problem Validation)
```
Skill A ──handoff.json──▶ Skill B ──handoff.json──▶ Skill C
```
Each skill reads only the prior handoff, not full history.

#### Pattern 2: Iterative Execution (Research Writing)
```
Skill A (iteration 1) ──state.json──▶ Skill A (iteration 2) ──state.json──▶ ...
```
Same skill executes repeatedly with state preserved in JSON.

#### Pattern 3: Parallel Execution (Multi-Source Research)
```
                    ┌──▶ Source A ──┐
Orchestrator ──────┼──▶ Source B ──┼──▶ Synthesis
                    └──▶ Source C ──┘
```
Multiple sources queried, results merged in synthesis step.

#### Pattern 4: Human-Gated Pipeline
```
Skill A ──▶ [USER APPROVAL] ──▶ Skill B ──▶ [USER APPROVAL] ──▶ Skill C
```
Explicit user confirmation required between phases.

### 5.5 Cross-Framework Compatibility

| Aspect | GSD | Ralph Loop | Academic Pipeline |
|--------|-----|------------|-------------------|
| State Format | Markdown + JSON | Append-only log + JSON | JSON state object |
| Iteration Tracking | iteration.txt | iteration.txt | iteration_count in state |
| Completion Signal | Phase status in ORCHESTRATION.md | Completion promise string | is_complete boolean |
| Human Checkpoints | gsd-phase-verifier | User confirmation | Interactive stages |
| Error Recovery | BLOCKED.md | BLOCKED.md | error status + retry |

**Interoperability:** All three systems use file-based state persistence and JSON handoffs, enabling cross-framework skill sharing

---

## 6. Design Philosophy

### 6.1 Core Thesis

**Context is the scarcest resource in LLM systems.** Every token consumed reduces the model's capacity for quality output. Traditional approaches treat context as unlimited, accumulating history until the system fails. GSD + Ralph Loop treats context as a fixed budget that must be actively managed.

### 6.2 The Seven Principles

#### Principle 1: File-Based State Persistence

**Definition:** All significant state is written to disk, not held in conversation memory.

**Rationale:** Conversation context is volatile, limited, and expensive. File system state is persistent, unlimited, and free to store.

**Implementation:**
- PROJECT.md captures vision and constraints
- ROADMAP.md captures phase structure
- STATE.md captures decisions and trade-offs
- PROGRESS.txt captures execution history
- handoffs/*.json captures inter-phase data

**Counter-Pattern (Avoid):** Relying on "the conversation so far" for state continuity.

---

#### Principle 2: Bounded Context Execution

**Definition:** Each iteration reads a fixed, small subset of available state (~3500 tokens).

**Rationale:** Attention quality degrades with context length. By bounding input, each iteration operates at peak attention.

**Implementation:**
- Hard token budgets per component
- Compression required for handoffs
- Pointers to files, not file contents
- Read last handoff only, not full history

**Counter-Pattern (Avoid):** Loading full project history before each operation.

---

#### Principle 3: Completion Promises

**Definition:** Each skill/phase has a machine-verifiable success marker.

**Rationale:** Without explicit completion signals, systems can't reliably advance or detect failure.

**Implementation:**
- `EXTRACTION_COMPLETE`, `RESEARCH_COMPLETE`, `REPORT_COMPLETE`
- Promises checked after each iteration
- No advancement until promise appears
- Max iterations prevent infinite loops

**Counter-Pattern (Avoid):** Vague success criteria like "when it feels done."

---

#### Principle 4: Summarize, Don't Transcribe

**Definition:** Handoffs contain compressed summaries, not raw outputs.

**Rationale:** 10,000 tokens of research output can be summarized to 500 tokens of actionable data. The next phase rarely needs the full output.

**Implementation:**
- Extract structured data first (JSON)
- Keep critical prose (decisions, next actions)
- Discard intermediate reasoning
- Point to full output files if needed

**Counter-Pattern (Avoid):** Passing full search results, API responses, or intermediate work products.

---

#### Principle 5: Human-in-the-Loop at Quality Gates

**Definition:** Humans verify completion, not AI self-assessment.

**Rationale:** AI systems can't reliably self-assess quality. Human verification catches issues before they compound.

**Implementation:**
- gsd-phase-verifier walks through UAT checklist
- User confirms each criterion
- Failure handling requires human decision
- No auto-advancement without approval

**Counter-Pattern (Avoid):** Auto-approval based on presence of outputs.

---

#### Principle 6: Two-Mode Planning (Discuss Then Plan)

**Definition:** Strategic discussion precedes formal planning.

**Rationale:** Plans made without discussion often miss critical context. Discussing first surfaces assumptions, trade-offs, and constraints.

**Implementation:**
- gsd-phase-planner Discuss mode captures strategy
- Decisions written to STATE.md
- Plan mode only runs after discussion
- Plans reference discussion decisions

**Counter-Pattern (Avoid):** Jumping directly to task breakdowns without strategic alignment.

---

#### Principle 7: Schema-Enforced Handoffs

**Definition:** Inter-phase communication uses typed JSON schemas with constraints.

**Rationale:** Unstructured handoffs drift, lose data, and resist compression. Schemas enforce completeness and bounds.

**Implementation:**
- maxLength constraints (200 chars typical)
- Required fields ensure nothing missing
- Enum types prevent invalid values
- Schema documented in templates

**Counter-Pattern (Avoid):** Free-form prose handoffs that grow unbounded.

---

### 6.3 Why These Principles Work Together

The principles form a coherent system:

1. **File-Based Persistence** enables **Bounded Context Execution** because state doesn't need to be in memory.

2. **Bounded Context Execution** requires **Summarize Don't Transcribe** because full outputs won't fit.

3. **Summarize Don't Transcribe** requires **Schema-Enforced Handoffs** to ensure nothing is lost.

4. **Schema-Enforced Handoffs** include **Completion Promises** as required fields.

5. **Completion Promises** enable **Human-in-the-Loop Verification** by providing clear success criteria.

6. **Human-in-the-Loop Verification** benefits from **Two-Mode Planning** because verification criteria come from discussed requirements.

7. **Two-Mode Planning** writes to **File-Based Persistence** (STATE.md), closing the loop.

### 6.4 The Fundamental Trade-Off

**GSD + Ralph Loop trades conversation fluidity for project completability.**

In a naive system, the AI maintains full conversational context—it remembers everything said, feels responsive, and requires no explicit state management. But this approach fails for complex projects.

In GSD + Ralph Loop, the AI operates statelessly within explicit boundaries. It reads only what's provided, writes explicit outputs, and depends on file system state. This feels less "magical" but enables arbitrarily complex projects.

**The trade-off is worth it when:**
- Projects span multiple sessions
- Projects have more than 3-4 phases
- Project state must survive context resets
- Quality must remain high through late phases
- Human verification is required at gates

**The trade-off is NOT worth it when:**
- Tasks complete in single interactions
- Conversational flow matters more than structure
- Overhead of state management exceeds benefit

### 6.5 Design Decisions and Their Rationale

| Decision | Rationale | Alternative Rejected |
|----------|-----------|---------------------|
| Append-only PROGRESS.txt | Creates audit trail, no data loss | Overwriting loses history |
| JSON handoffs (not YAML/XML) | Universal parsing, compact, typed | YAML ambiguous, XML verbose |
| Max iterations per phase | Prevents infinite loops | No limit risks runaway |
| Completion promises as strings | Simple grep verification | Complex completion logic |
| Two-mode planning | Separates strategy from tactics | Combined mode conflates |
| Interview-driven init | Captures human intent | Assumption-based scaffolding |
| UAT walk-through | Forces explicit verification | Auto-pass on file existence |
| Character limits in schemas | Enforces compression | Unbounded fields grow |

---

## 7. Gaps and Opportunities

### 7.1 Current Gaps in the Framework

#### Gap 1: No Automatic Context Compression

**Current State:** Token limits are hardcoded. When a handoff exceeds limits, it fails or truncates unpredictably.

**Desired State:** Automatic detection of approaching limits with intelligent compression.

**Opportunity:** Build a `context-compressor` skill that:
- Monitors token usage per component
- Triggers compression when approaching 80% of limit
- Uses extractive summarization for prose
- Preserves JSON structures while trimming arrays
- Logs compression decisions for debugging

---

#### Gap 2: Manual Token Counting

**Current State:** Developers must manually estimate token counts for handoffs and context windows.

**Desired State:** Built-in token counting with warnings.

**Opportunity:** Add token utilities to GSD framework:
- `count_tokens(text)` function in skill helpers
- Automatic warnings when handoffs exceed budgets
- Token usage dashboard in ORCHESTRATION.md
- Historical token trends in PROGRESS.txt

---

#### Gap 3: Limited Error Recovery Documentation

**Current State:** BLOCKED.md pattern exists but recovery procedures are sparse.

**Desired State:** Comprehensive error taxonomy with recovery playbooks.

**Opportunity:** Create error recovery guide:
- Taxonomy of failure modes (context overflow, skill failure, file system error, timeout)
- Recovery procedures for each mode
- Rollback mechanisms for partial phase completion
- Checkpoint/resume patterns for long-running phases

---

#### Gap 4: No Cross-Session State Resumption Standard

**Current State:** Each system (GSD, Ralph Loop, Academic Pipeline) has its own resumption approach.

**Desired State:** Unified resumption protocol.

**Opportunity:** Define `RESUME.md` standard:
- Canonical file that captures session termination state
- Required fields: last_phase, last_task, last_iteration, pending_actions
- Resumption protocol: read RESUME.md → validate state → continue
- Session boundary markers in PROGRESS.txt

---

#### Gap 5: No Skill Dependency Resolution

**Current State:** Skills are mapped to tasks manually in PLAN.md.

**Desired State:** Automatic skill dependency resolution.

**Opportunity:** Build skill registry:
- Metadata for each skill (inputs required, outputs produced)
- Automatic dependency graph generation
- Validation that required inputs exist before skill execution
- Suggested skill sequences based on task description

---

#### Gap 6: Limited Parallel Execution Support

**Current State:** Ralph Loop executes sequentially. Multi-source research happens in single iteration.

**Desired State:** Parallel execution with result merging.

**Opportunity:** Add parallel execution mode:
- Fork handoffs to multiple parallel branches
- Execute branches independently
- Merge results into unified handoff
- Handle partial failures (some branches succeed, some fail)

---

#### Gap 7: No Quality Metrics Beyond Pass/Fail

**Current State:** Verification is binary (pass/fail). No quality scoring.

**Desired State:** Continuous quality metrics.

**Opportunity:** Add quality scoring:
- Rubric-based scoring (1-10 per criterion)
- Trend tracking across iterations
- Quality regression detection
- Minimum quality thresholds for advancement

---

### 7.2 Enhancement Opportunities

#### Enhancement 1: Visual Pipeline Builder

**Current:** Pipelines defined in PLAN.md as YAML.

**Enhancement:** Graphical pipeline editor that generates PLAN.md.

**Value:** Reduces errors, improves visibility, enables non-technical users.

---

#### Enhancement 2: Skill Marketplace

**Current:** Skills are local files.

**Enhancement:** Registry of shareable, versioned skills.

**Value:** Community contribution, quality improvement, reduced duplication.

---

#### Enhancement 3: Real-Time Progress Dashboard

**Current:** Progress viewed by reading PROGRESS.txt and ORCHESTRATION.md.

**Enhancement:** Live dashboard showing:
- Current phase/task/iteration
- Token usage graphs
- Time elapsed per phase
- Predicted time to completion

**Value:** Project visibility, resource planning, stakeholder communication.

---

#### Enhancement 4: Automated Skill Generation

**Current:** Skills are manually authored.

**Enhancement:** Meta-skill that generates new skills from requirements.

**Value:** Faster skill development, consistent patterns, reduced expertise requirement.

---

#### Enhancement 5: Cross-Project Learning

**Current:** Each project starts fresh.

**Enhancement:** Learning system that captures patterns across projects:
- Which skill combinations work well together
- Optimal iteration counts by phase type
- Common compression patterns
- Failure mode frequencies

**Value:** Continuous improvement, reduced setup time, better defaults.

---

### 7.3 Research Questions

1. **Optimal Token Budget:** Is ~3500 tokens per iteration optimal, or could different budgets per skill type improve quality?

2. **Compression Quality:** How much information is lost in typical handoff compression? Can we measure "compression fidelity"?

3. **Human Checkpoint Frequency:** What's the optimal balance between human involvement and autonomous execution?

4. **Skill Granularity:** Should skills be fine-grained (single function) or coarse-grained (complete workflow)? What's the trade-off?

5. **Context Priming:** Would strategic pre-loading of key concepts improve iteration quality vs. loading fresh each time?

6. **Failure Prediction:** Can we predict which phases are likely to fail before executing them?

7. **Cross-Framework Portability:** How transferable are skills between GSD, Ralph Loop, and Academic Pipeline frameworks?

---

### 7.4 Recommended Next Steps

**Immediate (This Project):**
1. Document all current skills with standardized metadata
2. Create skill dependency graph
3. Build token counting utilities
4. Write error recovery playbook

**Short-Term (3-6 months):**
1. Implement automatic context compression
2. Create RESUME.md standard
3. Add quality scoring to verification
4. Build visual pipeline builder

**Long-Term (6-12 months):**
1. Skill marketplace infrastructure
2. Cross-project learning system
3. Parallel execution support
4. Automated skill generation

---

## Completion Status

- [x] Task 1.1: GSD Core Skills Analysis - `GSD_CORE_ANALYZED`
- [x] Task 1.2: Ralph Loop Patterns Analysis - `RALPH_LOOP_ANALYZED`
- [x] Task 1.3: Academic Pipeline Analysis - `ACADEMIC_PIPELINE_ANALYZED`
- [x] Task 1.4: Supporting Skills Analysis - `SUPPORTING_SKILLS_ANALYZED`
- [x] Task 1.5: Research Synthesis - `RESEARCH_SYNTHESIZED`

---

## Phase 1 Completion

**Promise:** `PHASE_1_DISCOVERY_COMPLETE`

**Deliverable:** This research document provides comprehensive analysis of:
- 6 GSD Core Skills with architecture, mechanics, and integration points
- Ralph Loop pattern with context management, compression strategy, and handoff schemas
- Academic Pipeline with 8-stage workflow and voice-first innovation
- 3 Supporting Skills with multi-perspective analysis
- Complete system integration map
- 7 design principles with rationale
- 7 gaps identified with enhancement opportunities
- Research questions and recommended next steps

**Word Count:** ~12,000 words
**Sections:** 7 major sections with 42 subsections
**Skills Analyzed:** 17 total (6 GSD + 1 Ralph orchestrator + 8 Academic + 3 Supporting)
