# Skill Dependencies: GSD + Ralph Loop Framework

**Project:** GSD Ralph
**Phase:** 3 - Gap Analysis & Skill Development
**Task:** 3.2 - Skill Dependencies
**Created:** 2026-01-19

---

## 1. GSD Orchestration Flow

How GSD skills invoke and depend on each other:

```mermaid
flowchart TB
    subgraph "User Commands"
        CMD["/gsd commands"]
    end

    subgraph "GSD Orchestration Layer"
        ORCH[gsd-orchestrator]
        INIT[gsd-init]
        PLANNER[gsd-phase-planner]
        EXECUTOR[gsd-phase-executor]
        VERIFIER[gsd-phase-verifier]
        STATE[gsd-state-manager]
    end

    subgraph "Canonical Files"
        PROJECT[PROJECT.md]
        ROADMAP[ROADMAP.md]
        ORCHESTRATION[ORCHESTRATION.md]
        STATED[STATE.md]
        PROGRESS[PROGRESS.txt]
        PLAN[PLAN.md]
        UAT[UAT.md]
    end

    CMD --> ORCH
    ORCH -->|"/gsd init"| INIT
    ORCH -->|"/gsd discuss-phase"| PLANNER
    ORCH -->|"/gsd plan-phase"| PLANNER
    ORCH -->|"/gsd execute-phase"| EXECUTOR
    ORCH -->|"/gsd verify-phase"| VERIFIER
    ORCH -->|"/gsd state"| STATE

    INIT -->|creates| PROJECT
    INIT -->|creates| ROADMAP
    INIT -->|creates| ORCHESTRATION
    INIT -->|creates| STATED
    INIT -->|creates| PROGRESS

    PLANNER -->|reads| ROADMAP
    PLANNER -->|reads/writes| STATED
    PLANNER -->|creates| PLAN
    PLANNER -->|creates| UAT

    EXECUTOR -->|reads| PLAN
    EXECUTOR -->|appends| PROGRESS
    EXECUTOR -->|updates| ORCHESTRATION

    VERIFIER -->|reads| UAT
    VERIFIER -->|updates| ROADMAP
    VERIFIER -->|updates| ORCHESTRATION

    STATE -->|reads/writes| STATED
```

---

## 2. GSD Lifecycle Sequence

The complete project lifecycle:

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant I as gsd-init
    participant P as gsd-phase-planner
    participant E as gsd-phase-executor
    participant V as gsd-phase-verifier
    participant S as gsd-state-manager

    U->>O: /gsd init
    O->>I: invoke
    I->>U: Interview questions
    U->>I: Responses
    I->>O: PROJECT_INITIALIZED

    loop For each phase
        U->>O: /gsd discuss-phase N
        O->>P: invoke (discuss mode)
        P->>U: Strategy questions
        U->>P: Decisions
        P->>S: Record decisions

        U->>O: /gsd plan-phase N
        O->>P: invoke (plan mode)
        P->>O: PHASE_PLANNED

        U->>O: /gsd execute-phase N
        O->>E: invoke
        E->>E: Ralph Loop iterations
        E->>O: PHASE_EXECUTED

        U->>O: /gsd verify-phase N
        O->>V: invoke
        V->>U: UAT walkthrough
        U->>V: Verification responses
        V->>O: PHASE_VERIFIED
    end
```

---

## 3. Ralph Loop Execution Pattern

How gsd-phase-executor implements bounded context iterations:

```mermaid
flowchart TB
    subgraph "Iteration N"
        LOAD[Load Context<br/>~3500 tokens]
        EXEC[Execute Skill]
        WRITE[Write Handoff<br/>~500 tokens]
        CHECK{Promise<br/>Met?}
    end

    subgraph "Context Components"
        PROJ[Project Summary<br/>300 tokens]
        TASK[Current Task<br/>500 tokens]
        HAND[Last Handoff<br/>500 tokens]
        SKILL[Skill Instructions<br/>1500 tokens]
    end

    subgraph "State Files"
        PROGRESS[PROGRESS.txt]
        HANDOFFS[handoffs/*.json]
        ITER[iteration.txt]
    end

    PROJ --> LOAD
    TASK --> LOAD
    HAND --> LOAD
    SKILL --> LOAD

    LOAD --> EXEC
    EXEC --> WRITE
    WRITE --> CHECK

    CHECK -->|No| LOAD
    CHECK -->|Yes| COMPLETE[Next Task]

    WRITE --> PROGRESS
    WRITE --> HANDOFFS
    WRITE --> ITER

    HANDOFFS --> HAND
```

---

## 4. Ralph Loop Pipeline (Problem Validation)

The 4-phase problem validation pipeline:

```mermaid
flowchart LR
    subgraph "Phase 1"
        P1[Orchestrator<br/>Assessment]
        H1[phase1-to-phase2.json]
    end

    subgraph "Phase 2"
        P2[problem-statement-<br/>extractor]
        H2[phase2-to-phase3.json]
    end

    subgraph "Phase 3"
        P3[problem-awareness-<br/>discovery]
        H3[phase3-to-phase4.json]
    end

    subgraph "Phase 4"
        P4[moxywolf<br/>+ docx]
        OUT[validation_report.docx]
    end

    INPUT[User Problem<br/>Description] --> P1
    P1 -->|ASSESSMENT_COMPLETE| H1
    H1 --> P2
    P2 -->|EXTRACTION_COMPLETE| H2
    H2 --> P3
    P3 -->|RESEARCH_COMPLETE| H3
    H3 --> P4
    P4 -->|REPORT_COMPLETE| OUT
```

**Handoff Token Flow:**

```mermaid
flowchart LR
    subgraph "Tokens Per Phase"
        T1["Phase 1<br/>In: 500 | Out: 400"]
        T2["Phase 2<br/>In: 900 | Out: 800"]
        T3["Phase 3<br/>In: 1300 | Out: 1200"]
        T4["Phase 4<br/>In: 2500 | Out: N/A"]
    end

    T1 --> T2 --> T3 --> T4
```

---

## 5. Academic Pipeline (8 Stages)

Complete academic content generation workflow:

```mermaid
flowchart TB
    subgraph "Input"
        BIB[BibTeX File]
    end

    subgraph "Stage 1: Analysis"
        S1[bibtex-theme-analyzer]
        O1[theme_analysis.json<br/>mermaid_diagram.md]
    end

    subgraph "Stage 2: Perspective"
        S2[academic-perspective-builder]
        O2[perspective.json]
    end

    subgraph "Stage 3: Voice"
        S3[voice-injection]
        O3[voice_context.json]
    end

    subgraph "Stage 4: Format"
        S4[academia-formatting]
        O4[formatting_requirements.json]
    end

    subgraph "Stage 5: Structure"
        S5[research-analyst]
        O5[handoff_for_writer.json]
    end

    subgraph "Stage 6: Writing"
        S6[research-writer]
        O6[accumulated_content]
    end

    subgraph "Stage 7: Bibliography"
        S7[bibliography-generator]
        O7[complete_document.md]
    end

    subgraph "Stage 8: Critique"
        S8[professor]
        O8[critique_report.md<br/>improvement_plan.md]
    end

    BIB --> S1
    S1 --> O1 --> S2
    S2 --> O2 --> S3
    S3 --> O3 --> S4
    S4 --> O4 --> S5
    O2 --> S5
    O3 --> S5
    S5 --> O5 --> S6
    S6 --> O6 --> S7
    S7 --> O7 --> S8
    S8 --> O8
```

**Key Innovation Highlighted:**

```mermaid
flowchart LR
    subgraph "Traditional"
        T1[Write Content] --> T2[Add Voice] --> T3[Format]
    end

    subgraph "Academic Pipeline"
        A1[Capture Voice] --> A2[Define Format] --> A3[Plan with Voice] --> A4[Write with Voice Baked In]
    end

    style A1 fill:#F77028
    style A2 fill:#F77028
```

---

## 6. Cross-Framework Integration

How GSD orchestrates domain skills:

```mermaid
flowchart TB
    subgraph "GSD Layer"
        GSDO[gsd-orchestrator]
        GSDP[gsd-phase-planner]
        GSDE[gsd-phase-executor]
        GSDV[gsd-phase-verifier]
    end

    subgraph "Ralph Loop Pattern"
        RALPH[Ralph Loop<br/>Bounded Context]
    end

    subgraph "Domain Pipelines"
        PROB[Problem Validation<br/>Pipeline]
        ACAD[Academic<br/>Pipeline]
        CUSTOM[Custom<br/>Pipelines]
    end

    subgraph "Individual Skills"
        PSE[problem-statement-extractor]
        PAD[problem-awareness-discovery]
        MOXY[moxywolf]
        RA[research-analyst]
        RW[research-writer]
        BG[bibliography-generator]
    end

    GSDO --> GSDP
    GSDP --> GSDE
    GSDE --> RALPH
    RALPH --> GSDV

    RALPH --> PROB
    RALPH --> ACAD
    RALPH --> CUSTOM

    PROB --> PSE
    PROB --> PAD
    PROB --> MOXY

    ACAD --> RA
    ACAD --> RW
    ACAD --> BG
    ACAD --> MOXY
```

---

## 7. Skill Invocation Matrix

Which skills can invoke which other skills:

```mermaid
flowchart LR
    subgraph "Orchestrators"
        O1[gsd-orchestrator]
        O2[ralph-problem-validation-orchestrator]
        O3[academic-pipeline-orchestrator]
    end

    subgraph "Can Invoke"
        direction TB
        G1[gsd-init]
        G2[gsd-phase-planner]
        G3[gsd-phase-executor]
        G4[gsd-phase-verifier]
        G5[gsd-state-manager]

        D1[problem-statement-extractor]
        D2[problem-awareness-discovery]
        D3[moxywolf]

        A1[bibtex-theme-analyzer]
        A2[academic-perspective-builder]
        A3[voice-injection]
        A4[research-analyst]
        A5[research-writer]
        A6[bibliography-generator]
        A7[professor]
    end

    O1 --> G1
    O1 --> G2
    O1 --> G3
    O1 --> G4
    O1 --> G5

    O2 --> D1
    O2 --> D2
    O2 --> D3

    O3 --> A1
    O3 --> A2
    O3 --> A3
    O3 --> A4
    O3 --> A5
    O3 --> A6
    O3 --> A7
    O3 --> D3
```

---

## 8. Dependency Summary Table

| Skill | Depends On | Depended By |
|-------|------------|-------------|
| gsd-init | - | All GSD skills |
| gsd-orchestrator | gsd-init | - |
| gsd-phase-planner | gsd-init | gsd-phase-executor |
| gsd-phase-executor | gsd-phase-planner | gsd-phase-verifier |
| gsd-phase-verifier | gsd-phase-executor | - |
| gsd-state-manager | gsd-init | gsd-phase-planner |
| problem-statement-extractor | - | problem-awareness-discovery |
| problem-awareness-discovery | problem-statement-extractor | moxywolf (in pipeline) |
| moxywolf | - | Multiple (terminal skill) |
| research-analyst | perspective-builder, voice-injection | research-writer |
| research-writer | research-analyst | bibliography-generator |
| bibliography-generator | research-writer | professor |
| professor | bibliography-generator | - |

---

## Completion Status

**Task 3.2:** Skill Dependencies - `DEPENDENCY_GRAPH_COMPLETE`

**Diagrams Created:** 8
**Formats:** Mermaid flowcharts and sequence diagrams
**Coverage:** GSD lifecycle, Ralph Loop, Academic Pipeline, Cross-framework

