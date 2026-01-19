# Skill Registry: GSD + Ralph Loop Framework

**Project:** GSD Ralph
**Phase:** 3 - Gap Analysis & Skill Development
**Task:** 3.1 - Skill Registry
**Created:** 2026-01-19

---

## Registry Overview

This registry documents all skills in the GSD + Ralph Loop ecosystem with standardized metadata enabling dependency resolution, token budgeting, and integration planning.

**Total Skills:** 17
**Categories:** 5 (Orchestration, Execution, Verification, State, Domain)

---

## Metadata Schema

```yaml
skill_name: string
version: string
purpose: string (1-2 sentences)
category: enum [orchestration, execution, verification, state, domain]
inputs:
  - name: string
    type: enum [file, json, user_input, prior_handoff]
    required: boolean
outputs:
  - name: string
    type: enum [file, json, handoff, deliverable]
completion_promise: string
token_budget:
  typical: number
  max: number
dependencies: [skill_names]
human_interaction: enum [none, optional, required]
```

---

## Category 1: GSD Orchestration Skills

### 1.1 gsd-init

```yaml
skill_name: gsd-init
version: "1.0"
purpose: Bootstrap new GSD projects through interactive interview, creating canonical file structure.
category: orchestration

inputs:
  - name: project_name
    type: user_input
    required: true
  - name: vision_responses
    type: user_input
    required: true
  - name: constraints_responses
    type: user_input
    required: true

outputs:
  - name: PROJECT.md
    type: file
  - name: REQUIREMENTS.md
    type: file
  - name: ROADMAP.md
    type: file
  - name: ORCHESTRATION.md
    type: file
  - name: STATE.md
    type: file
  - name: PROGRESS.txt
    type: file
  - name: iteration.txt
    type: file

completion_promise: "PROJECT_INITIALIZED"

token_budget:
  typical: 2000
  max: 3000

dependencies: []

human_interaction: required
```

---

### 1.2 gsd-orchestrator

```yaml
skill_name: gsd-orchestrator
version: "1.0"
purpose: Meta-router that dispatches /gsd commands to appropriate specialized skills.
category: orchestration

inputs:
  - name: user_command
    type: user_input
    required: true
  - name: ORCHESTRATION.md
    type: file
    required: true
  - name: PROJECT.md
    type: file
    required: false

outputs:
  - name: skill_invocation
    type: json
  - name: ORCHESTRATION.md
    type: file

completion_promise: "COMMAND_ROUTED"

token_budget:
  typical: 500
  max: 1000

dependencies:
  - gsd-init (creates required files)

human_interaction: none
```

---

### 1.3 gsd-phase-planner

```yaml
skill_name: gsd-phase-planner
version: "1.0"
purpose: Two-mode skill for phase planning - discuss strategy then create formal plan.
category: orchestration

inputs:
  - name: ROADMAP.md
    type: file
    required: true
  - name: STATE.md
    type: file
    required: true
  - name: phase_number
    type: user_input
    required: true
  - name: discussion_responses
    type: user_input
    required: true (discuss mode)

outputs:
  - name: phases/phase-N/PLAN.md
    type: file
  - name: phases/phase-N/UAT.md
    type: file
  - name: STATE.md (updated)
    type: file

completion_promise: "PHASE_PLANNED"

token_budget:
  typical: 1500
  max: 2500

dependencies:
  - gsd-init

human_interaction: required (discuss mode), optional (plan mode)
```

---

## Category 2: GSD Execution Skills

### 2.1 gsd-phase-executor

```yaml
skill_name: gsd-phase-executor
version: "1.0"
purpose: Execute phase tasks using Ralph Loop pattern with bounded context iterations.
category: execution

inputs:
  - name: phases/phase-N/PLAN.md
    type: file
    required: true
  - name: handoffs/last_handoff.json
    type: prior_handoff
    required: false
  - name: skill_instructions
    type: file
    required: true

outputs:
  - name: PROGRESS.txt (appended)
    type: file
  - name: handoffs/phase-N-task-M.json
    type: handoff
  - name: phases/phase-N/SUMMARY.md
    type: file
  - name: task_outputs
    type: deliverable

completion_promise: "PHASE_EXECUTED"

token_budget:
  typical: 3500
  max: 4000

dependencies:
  - gsd-phase-planner

human_interaction: optional (check-ins configurable)
```

---

## Category 3: GSD Verification Skills

### 3.1 gsd-phase-verifier

```yaml
skill_name: gsd-phase-verifier
version: "1.0"
purpose: Interactive UAT walk-through to verify phase completion against criteria.
category: verification

inputs:
  - name: phases/phase-N/SUMMARY.md
    type: file
    required: true
  - name: phases/phase-N/UAT.md
    type: file
    required: true
  - name: user_verification_responses
    type: user_input
    required: true

outputs:
  - name: phases/phase-N/VERIFICATION.md
    type: file
  - name: ROADMAP.md (status updated)
    type: file
  - name: ORCHESTRATION.md (status updated)
    type: file

completion_promise: "PHASE_VERIFIED"

token_budget:
  typical: 1000
  max: 1500

dependencies:
  - gsd-phase-executor

human_interaction: required
```

---

## Category 4: GSD State Skills

### 4.1 gsd-state-manager

```yaml
skill_name: gsd-state-manager
version: "1.0"
purpose: Manage STATE.md for tracking decisions, trade-offs, questions, and learnings.
category: state

inputs:
  - name: STATE.md
    type: file
    required: true
  - name: mode
    type: user_input
    required: true
  - name: state_content
    type: user_input
    required: true (for write modes)

outputs:
  - name: STATE.md (updated)
    type: file
  - name: state_report
    type: json

completion_promise: "STATE_UPDATED"

token_budget:
  typical: 500
  max: 1000

dependencies:
  - gsd-init

human_interaction: optional
```

---

## Category 5: Domain Skills

### 5.1 ralph-problem-validation-orchestrator

```yaml
skill_name: ralph-problem-validation-orchestrator
version: "1.0"
purpose: Orchestrate 4-phase problem validation using Ralph Loop pattern.
category: domain

inputs:
  - name: user_problem_description
    type: user_input
    required: true
  - name: context_information
    type: user_input
    required: false

outputs:
  - name: ORCHESTRATION.md
    type: file
  - name: PROGRESS.txt
    type: file
  - name: handoffs/*.json
    type: handoff
  - name: validation_report.docx
    type: deliverable

completion_promise: "VALIDATION_COMPLETE"

token_budget:
  typical: 3500
  max: 4000

dependencies: []

human_interaction: optional (user confirmation between phases)

sub_skills:
  - problem-statement-extractor
  - problem-awareness-discovery
  - moxywolf
```

---

### 5.2 problem-statement-extractor

```yaml
skill_name: problem-statement-extractor
version: "1.0"
purpose: Extract rigorous problem statements using multi-persona swarm analysis.
category: domain

inputs:
  - name: analysis_content
    type: prior_handoff
    required: true
  - name: output_format
    type: user_input
    required: false

outputs:
  - name: problem_statement
    type: json
  - name: validation_checklist
    type: json

completion_promise: "EXTRACTION_COMPLETE"

token_budget:
  typical: 1000
  max: 1500

dependencies: []

human_interaction: required (user confirms final statement)
```

---

### 5.3 problem-awareness-discovery

```yaml
skill_name: problem-awareness-discovery
version: "1.0"
purpose: Validate problem hypothesis via multi-source market research.
category: domain

inputs:
  - name: problem_statement
    type: prior_handoff
    required: true
  - name: searchable_elements
    type: prior_handoff
    required: true

outputs:
  - name: validation_rating
    type: json
  - name: icp_profile
    type: json
  - name: key_findings
    type: json
  - name: language_gold
    type: json
  - name: research_report.md
    type: deliverable

completion_promise: "RESEARCH_COMPLETE"

token_budget:
  typical: 2000
  max: 3000

dependencies:
  - problem-statement-extractor

human_interaction: none

external_tools:
  - COMPOSIO_SEARCH_WEB
  - COMPOSIO_SEARCH_TRENDS
  - COMPOSIO_SEARCH_NEWS
  - REDDIT_SEARCH_ACROSS_SUBREDDITS
```

---

### 5.4 moxywolf

```yaml
skill_name: moxywolf
version: "1.0"
purpose: Apply MoxyWolf brand guidelines for writing, visuals, and document styling.
category: domain

inputs:
  - name: content_to_style
    type: prior_handoff
    required: true
  - name: document_type
    type: user_input
    required: false

outputs:
  - name: styled_document
    type: deliverable
  - name: brand_checklist
    type: json

completion_promise: "CONTENT_COMPLETE"

token_budget:
  typical: 1500
  max: 2500

dependencies: []

human_interaction: optional (rewrite test)
```

---

### 5.5 academic-pipeline-orchestrator

```yaml
skill_name: academic-pipeline-orchestrator
version: "1.0"
purpose: Orchestrate 8-stage academic content generation from bibliography to critique.
category: domain

inputs:
  - name: bibtex_content
    type: file
    required: true
  - name: perspective_responses
    type: user_input
    required: true
  - name: voice_responses
    type: user_input
    required: true

outputs:
  - name: .pipeline_state.json
    type: file
  - name: complete_document.md
    type: deliverable
  - name: critique_report.md
    type: deliverable
  - name: improvement_plan.md
    type: deliverable

completion_promise: "PIPELINE_COMPLETE"

token_budget:
  typical: 3500
  max: 4000

dependencies: []

human_interaction: required (stages 2, 3, 5)

sub_skills:
  - bibtex-theme-analyzer
  - academic-perspective-builder
  - voice-injection
  - academia-formatting
  - research-analyst
  - research-writer
  - bibliography-generator
  - professor
```

---

### 5.6 research-analyst

```yaml
skill_name: research-analyst
version: "1.0"
purpose: Analyze sources, create structure plan, select citations for academic writing.
category: domain

inputs:
  - name: perspective.json
    type: prior_handoff
    required: true
  - name: voice_context.json
    type: prior_handoff
    required: true
  - name: bibtex_content
    type: file
    required: true

outputs:
  - name: handoff_for_writer.json
    type: handoff
  - name: structure_plan
    type: json

completion_promise: "ANALYSIS_COMPLETE"

token_budget:
  typical: 1500
  max: 2000

dependencies:
  - academic-perspective-builder
  - voice-injection

human_interaction: required (structure approval)
```

---

### 5.7 research-writer

```yaml
skill_name: research-writer
version: "1.0"
purpose: Write academic content iteratively, one section at a time with citations.
category: domain

inputs:
  - name: handoff_for_writer.json
    type: prior_handoff
    required: true
  - name: writer_state.json
    type: prior_handoff
    required: true
  - name: bibtex_content
    type: file
    required: true

outputs:
  - name: section_content
    type: json
  - name: writer_state.json (updated)
    type: handoff
  - name: accumulated_content
    type: deliverable

completion_promise: "SECTION_COMPLETE" (per section) / "WRITING_COMPLETE" (final)

token_budget:
  typical: 2000
  max: 2500

dependencies:
  - research-analyst

human_interaction: none
```

---

### 5.8 bibliography-generator

```yaml
skill_name: bibliography-generator
version: "1.0"
purpose: Generate properly formatted bibliography from citations in specified style.
category: domain

inputs:
  - name: all_citations_used
    type: prior_handoff
    required: true
  - name: citation_style
    type: user_input
    required: false
  - name: bibtex_content
    type: file
    required: true

outputs:
  - name: formatted_bibliography
    type: deliverable
  - name: complete_document.md
    type: deliverable

completion_promise: "BIBLIOGRAPHY_COMPLETE"

token_budget:
  typical: 1000
  max: 1500

dependencies:
  - research-writer

human_interaction: none
```

---

### 5.9 professor

```yaml
skill_name: professor
version: "1.0"
purpose: Comprehensive 10-phase academic paper critique with improvement plan.
category: domain

inputs:
  - name: complete_document.md
    type: file
    required: true
  - name: bibtex_content
    type: file
    required: true

outputs:
  - name: critique_report.md
    type: deliverable
  - name: improvement_plan.md
    type: deliverable
  - name: scores
    type: json

completion_promise: "CRITIQUE_COMPLETE"

token_budget:
  typical: 2000
  max: 3000

dependencies:
  - bibliography-generator

human_interaction: none
```

---

## Summary Tables

### Skills by Category

| Category | Count | Skills |
|----------|-------|--------|
| Orchestration | 3 | gsd-init, gsd-orchestrator, gsd-phase-planner |
| Execution | 1 | gsd-phase-executor |
| Verification | 1 | gsd-phase-verifier |
| State | 1 | gsd-state-manager |
| Domain | 11 | ralph-*, problem-*, moxywolf, academic-*, research-*, bibliography-*, professor |

### Skills by Human Interaction

| Interaction | Skills |
|-------------|--------|
| Required | gsd-init, gsd-phase-planner (discuss), gsd-phase-verifier, problem-statement-extractor, academic-pipeline-orchestrator, research-analyst |
| Optional | gsd-phase-executor, gsd-state-manager, ralph-problem-validation-orchestrator, moxywolf |
| None | gsd-orchestrator, problem-awareness-discovery, research-writer, bibliography-generator, professor |

### Token Budgets

| Budget Range | Skills |
|--------------|--------|
| 500-1000 | gsd-orchestrator, gsd-state-manager, problem-statement-extractor, bibliography-generator |
| 1000-2000 | gsd-phase-planner, gsd-phase-verifier, research-analyst, moxywolf |
| 2000-3000 | gsd-init, problem-awareness-discovery, research-writer, professor |
| 3000-4000 | gsd-phase-executor, ralph-problem-validation-orchestrator, academic-pipeline-orchestrator |

---

## Completion Status

**Task 3.1:** Skill Registry - `SKILL_REGISTRY_COMPLETE`

**Skills Documented:** 17
**Categories:** 5
**Metadata Fields:** 10 per skill

