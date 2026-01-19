# Paper Outline: Bounded Context Orchestration for LLM Workflows

**Project:** GSD Ralph
**Phase:** 2 - Theoretical Framework
**Task:** 2.4 - Paper Outline
**Created:** 2026-01-19
**Target Length:** 8,000-10,000 words

---

## Paper Metadata

**Title:** Bounded Context Orchestration: A Framework for Multi-Phase LLM Workflows Without Context Exhaustion

**Subtitle:** How File-Based State Persistence and Schema-Enforced Handoffs Enable Arbitrarily Complex AI Workflows

**Authors:** Dorian Crutcher (MoxyWolf LLC)

**Keywords:** Large Language Models, Context Management, Workflow Orchestration, State Persistence, Multi-Agent Systems, Prompt Engineering

---

## Section Outline

### Abstract (200-250 words)

**Content:**
- Problem: Context exhaustion in multi-phase LLM workflows
- Gap: No existing methodology addresses bounded context with file-based persistence
- Solution: Bounded Context Orchestration (BCO) framework
- Method: GSD (Get Shit Done) + Ralph Loop implementation
- Results: Quality maintained across arbitrary phase counts
- Contribution: Seven principles forming coherent theoretical system

**Key Claims:**
1. Context is the scarcest resource in LLM systems
2. Bounded execution maintains attention quality
3. File-based persistence enables multi-session workflows
4. Schema-enforced handoffs prevent information loss
5. Framework enables complex workflows previously infeasible

---

### 1. Introduction (800-1000 words)

#### 1.1 Context and Motivation (300 words)
- LLMs enable complex AI applications
- Multi-phase workflows are increasingly common
- Context accumulation is an overlooked problem
- Quality degradation is often undetected until too late

#### 1.2 The Problem (300 words)
- Formal definition of context exhaustion
- Example: 6-phase workflow degradation
- Why existing solutions (longer context, RAG) don't fully address it

#### 1.3 Our Contribution (200 words)
- Introduce Bounded Context Orchestration (BCO)
- Present GSD + Ralph Loop as implementation
- Articulate seven design principles
- Demonstrate with self-referential case study

#### 1.4 Paper Structure (100 words)
- Roadmap of remaining sections

---

### 2. Background and Related Work (1200-1500 words)

#### 2.1 Transformer Context Limitations (400 words)
- Quadratic attention complexity
- Lost-in-the-middle phenomenon
- Effective context vs. stated context
- Key citations: Liu et al. 2023, YARN, Mamba

#### 2.2 Existing Approaches (500 words)

##### 2.2.1 Long-Context Models
- Increasing C_max
- Still suffers attention degradation

##### 2.2.2 Retrieval-Augmented Generation
- What to include
- Doesn't bound total context

##### 2.2.3 Memory Systems
- External memory approaches
- Heavy infrastructure requirements

##### 2.2.4 Multi-Agent Systems
- Orchestrator-worker patterns
- Context per agent undefined

#### 2.3 Prompt Chaining (300 words)
- Sequential prompt approaches
- Lack of state persistence
- No compression strategy
- Key citations: ACL 2024, IUI 2024

#### 2.4 Gap Identification (200 words)
- What's missing: bounded context + file-based persistence + compression
- Why this matters for production systems

---

### 3. Theoretical Framework (1500-1800 words)

#### 3.1 Problem Formalization (400 words)
- Mathematical model of context exhaustion
- Define C_max, C_eff, T_i, Q(c)
- Show context accumulation formula
- Define the problem precisely

#### 3.2 The Bounded Context Solution (400 words)
- Core insight: stateless function with bounded input
- Define bounded context budget B
- Show context stays bounded across phases
- Define compression function φ

#### 3.3 The Seven Principles (600 words)

##### 3.3.1 File-Based State Persistence
- Addresses: context volatility
- Implementation: canonical file structure

##### 3.3.2 Bounded Context Execution
- Addresses: attention degradation
- Implementation: hard token budgets

##### 3.3.3 Completion Promises
- Addresses: ambiguous completion
- Implementation: machine-verifiable markers

##### 3.3.4 Summarize, Don't Transcribe
- Addresses: handoff bloat
- Implementation: compression strategies

##### 3.3.5 Human-in-the-Loop at Quality Gates
- Addresses: quality drift
- Implementation: UAT verification

##### 3.3.6 Two-Mode Planning
- Addresses: misaligned plans
- Implementation: discuss then plan

##### 3.3.7 Schema-Enforced Handoffs
- Addresses: handoff drift
- Implementation: typed JSON schemas

#### 3.4 Principle Interdependence (200 words)
- Causal chain between principles
- Why they form coherent system
- What breaks if one is violated

---

### 4. Implementation: GSD + Ralph Loop (1500-1800 words)

#### 4.1 System Architecture (400 words)
- Overview diagram
- Three layers: orchestration, execution, state
- Component descriptions

#### 4.2 GSD Framework (500 words)

##### 4.2.1 Core Skills
- gsd-init, gsd-orchestrator, gsd-phase-planner
- gsd-phase-executor, gsd-phase-verifier, gsd-state-manager

##### 4.2.2 Canonical File Structure
- PROJECT.md, ROADMAP.md, STATE.md
- PROGRESS.txt, handoffs/*.json

##### 4.2.3 Workflow
- Initialize → Plan → Execute → Verify cycle

#### 4.3 Ralph Loop Pattern (500 words)

##### 4.3.1 Stateless Iteration
- Read bounded context (~3500 tokens)
- Execute skill
- Write compressed handoff (~500 tokens)

##### 4.3.2 Token Budget Breakdown
- Project summary: 300 tokens
- Current task: 500 tokens
- Last handoff: 500 tokens
- Skill instructions: 1500 tokens
- Buffer: 700 tokens

##### 4.3.3 Compression Examples
- Before/after handoff comparison
- JSON schema enforcement

#### 4.4 Integration Points (200 words)
- How GSD orchestrates Ralph Loop
- How skills are invoked
- How state flows between components

---

### 5. Evaluation (1200-1500 words)

#### 5.1 Case Study: This Paper (600 words)

##### 5.1.1 Methodology
- Self-referential demonstration
- 6-phase project structure
- Phase descriptions and deliverables

##### 5.1.2 Execution Metrics
- Phases completed: 6
- Total iterations: ~50
- Context per iteration: ~3500 tokens
- Files generated: 20+

##### 5.1.3 Quality Observations
- No degradation in later phases
- Consistent voice and structure
- Complete deliverables at each phase

#### 5.2 Token Usage Analysis (400 words)

##### 5.2.1 Naive vs. BCO Comparison
- Naive: 4K → 12K → 27K → 39K+ (unbounded growth)
- BCO: 3.5K → 3.5K → 3.5K → 3.5K (bounded)

##### 5.2.2 Compression Effectiveness
- Input: ~10K tokens of phase output
- Handoff: ~500-1000 tokens
- Compression ratio: 5-10x

#### 5.3 Qualitative Assessment (300 words)
- Debuggability: file inspection vs. conversation replay
- Auditability: complete execution trail
- Portability: no vendor lock-in
- Session continuity: automatic state resumption

---

### 6. Discussion (800-1000 words)

#### 6.1 Implications (300 words)
- For practitioners: lightweight orchestration without heavy infrastructure
- For researchers: new dimension of LLM system design
- For industry: inspectable, auditable AI workflows

#### 6.2 Limitations (300 words)
- Overhead for simple tasks
- Manual schema definition
- Sequential execution (limited parallelism)
- Approximate token counting

#### 6.3 Future Work (300 words)
- Automatic schema generation
- Adaptive token budgets
- Parallel bounded execution
- Cross-session state merging
- Formal verification of principle adherence

---

### 7. Conclusion (300-400 words)

#### 7.1 Summary (150 words)
- Restate problem, solution, contribution
- Key results

#### 7.2 Takeaways (150 words)
- Context is the scarcest LLM resource
- Bounded execution maintains quality
- Seven principles form coherent system
- Simple file-based approach is effective

---

### References (Not counted in word limit)

- 15+ academic sources from literature review
- Proper BibTeX formatting
- DOIs where available

---

### Appendices (Optional, not counted)

#### A. Complete File Structure
- Full canonical file listing

#### B. Schema Examples
- Handoff JSON schemas with examples

#### C. Skill Summaries
- One-paragraph description of each GSD skill

---

## Word Count Targets

| Section | Target Words | % of Paper |
|---------|--------------|------------|
| Abstract | 225 | 2.5% |
| 1. Introduction | 900 | 10% |
| 2. Background | 1350 | 15% |
| 3. Theory | 1650 | 18% |
| 4. Implementation | 1650 | 18% |
| 5. Evaluation | 1350 | 15% |
| 6. Discussion | 900 | 10% |
| 7. Conclusion | 350 | 4% |
| **Total** | **8375** | **~93%** |
| Buffer/Expansion | 625 | 7% |
| **Grand Total** | **9000** | 100% |

---

## Key Arguments Mapped to Sections

| Argument | Primary Section | Supporting Sections |
|----------|-----------------|---------------------|
| Context exhaustion is real problem | 1, 2.1 | 3.1, 5.2 |
| Existing solutions insufficient | 2.2-2.4 | 3.2 |
| Seven principles form coherent system | 3.3, 3.4 | 4.2, 4.3 |
| File-based persistence is sufficient | 3.3.1, 4.2.2 | 5.3 |
| Bounded context maintains quality | 3.2, 3.3.2 | 5.1, 5.2 |
| Framework enables complex workflows | 4, 5.1 | 6.1 |
| Approach is practical and lightweight | 5.3, 6.1 | 4.1 |

---

## Completion Status

**Task 2.4:** Paper Outline - `OUTLINE_COMPLETE`

**Deliverables:**
- Complete section-by-section outline
- Word count targets per section
- Key arguments mapped to sections
- Appendix structure defined

