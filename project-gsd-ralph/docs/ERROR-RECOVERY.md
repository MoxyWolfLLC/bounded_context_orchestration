# Error Recovery Playbook: GSD + Ralph Loop Framework

**Project:** GSD Ralph
**Phase:** 3 - Gap Analysis & Skill Development
**Task:** 3.4 - Error Recovery Playbook
**Created:** 2026-01-19

---

## Quick Reference

| Failure Type | Detection | Immediate Action | Recovery |
|--------------|-----------|------------------|----------|
| Context Overflow | Token count > budget | Stop, compress | Checkpoint and resume |
| Skill Failure | No completion promise | Log, diagnose | Retry or re-plan |
| Schema Violation | JSON validation fails | Identify field | Fix and re-submit |
| File System Error | Read/write fails | Check permissions | Use backup location |
| Human Gate Failure | UAT criteria fails | Document severity | Fix, re-plan, or accept |
| Timeout | Iteration exceeds limit | Force stop | Reduce scope or split |

---

## 1. Context Overflow

### 1.1 Detection

**Symptoms:**
- Output quality degrades noticeably
- Model produces repetitive or circular content
- Key information from early context is forgotten
- Iteration takes unusually long

**Measurement:**
- Handoff exceeds 500-token target
- Total context approaches 4000+ tokens
- PROGRESS.txt entries become longer over time

### 1.2 Immediate Actions

1. **Stop execution** – Don't continue accumulating context
2. **Log current state** – Record what was completed
3. **Identify overflow source** – Which component exceeded budget?

### 1.3 Recovery Procedure

```
1. Create CHECKPOINT.md:
   - Current phase/task/iteration
   - What was completed
   - What remains
   - Last valid handoff reference

2. Compress the overflowing component:
   - Extract key-value pairs from prose
   - Remove intermediate reasoning
   - Keep only conclusions and file paths
   - Verify compressed version < 500 tokens

3. Update handoff with compressed version

4. Resume from checkpoint:
   - Load compressed handoff
   - Continue with next task
```

### 1.4 Prevention

- **Monitor token counts** – Estimate before execution
- **Use schema constraints** – maxLength fields enforce compression
- **Read last handoff only** – Never load full PROGRESS.txt
- **Prefer pointers** – File paths instead of file contents

---

## 2. Skill Failure

### 2.1 Detection

**Symptoms:**
- Completion promise doesn't appear in output
- Output format doesn't match expected schema
- Skill produces error messages
- Iteration count reaches maximum without completion

### 2.2 Immediate Actions

1. **Check output** – Is the promise present anywhere?
2. **Verify inputs** – Were required inputs provided?
3. **Review skill instructions** – Were they loaded correctly?

### 2.3 Recovery Procedure

```
1. Diagnose failure type:
   a. Input missing → Provide missing input, retry
   b. Instructions unclear → Clarify prompt, retry
   c. Skill bug → Document bug, use workaround
   d. Task too complex → Break into subtasks

2. If retryable:
   - Log failure reason
   - Adjust inputs or instructions
   - Retry iteration (don't increment counter yet)

3. If not retryable:
   - Create BLOCKED.md with:
     - Task that failed
     - Failure reason
     - What's needed to unblock
   - Update ORCHESTRATION.md status to "blocked"
   - Notify user
```

### 2.4 Prevention

- **Validate inputs** before skill execution
- **Set realistic max_iterations** per task type
- **Include fallback instructions** in skill docs
- **Test skills** before using in pipelines

---

## 3. Schema Violation

### 3.1 Detection

**Symptoms:**
- JSON parsing fails
- Required fields missing
- Field values outside allowed ranges
- Type mismatches (string where number expected)

### 3.2 Immediate Actions

1. **Identify violation** – Which field failed validation?
2. **Check source** – Was it the skill output or handoff corruption?
3. **Preserve original** – Don't overwrite potentially recoverable data

### 3.3 Recovery Procedure

```
1. Locate the schema (in /templates/handoff-schemas/)

2. Compare output against schema:
   - List missing required fields
   - Identify type mismatches
   - Check length constraints

3. Fix the handoff:
   a. If field missing → Add with default or extracted value
   b. If wrong type → Convert or regenerate
   c. If too long → Compress to fit maxLength

4. Validate fixed handoff against schema

5. Continue execution with fixed handoff
```

### 3.4 Prevention

- **Use schema validation** in handoff writing code
- **Include examples** in skill instructions
- **Fail fast** – Validate immediately after generation
- **Version schemas** – Don't change schemas mid-project

---

## 4. File System Error

### 4.1 Detection

**Symptoms:**
- "File not found" errors
- "Permission denied" errors
- Disk full errors
- Corrupted file content

### 4.2 Immediate Actions

1. **Identify affected file** – Which read/write failed?
2. **Check file system state** – Does file exist? Permissions?
3. **Assess impact** – Is this blocking or can work continue?

### 4.3 Recovery Procedure

```
1. For read errors:
   a. Check file path (typos, case sensitivity)
   b. Check file exists (ls -la)
   c. Check permissions (chmod if needed)
   d. Use backup if available (handoffs/ has copies)

2. For write errors:
   a. Check directory exists (mkdir -p)
   b. Check disk space (df -h)
   c. Check permissions
   d. Use alternate location if needed

3. For corruption:
   a. Identify last good version (git history or backups)
   b. Restore from PROGRESS.txt (append-only)
   c. Regenerate if needed
```

### 4.4 Prevention

- **Use absolute paths** – Avoid ambiguous relative paths
- **Create directories** before writing files
- **Append, don't overwrite** – PROGRESS.txt pattern
- **Backup critical files** – Copy before modifying

---

## 5. Human Gate Failure

### 5.1 Detection

**Symptoms:**
- User marks UAT criterion as "no"
- Verification walkthrough identifies issues
- Quality doesn't meet requirements
- Deliverables incomplete or incorrect

### 5.2 Immediate Actions

1. **Document failure** – Which criterion failed? Why?
2. **Assess severity** – Critical, Major, or Minor?
3. **Gather context** – What would fix it?

### 5.3 Recovery Procedure

```
1. Classify severity:
   - CRITICAL: Must fix before proceeding
   - MAJOR: Should fix, could proceed with risk
   - MINOR: Nice to have, can proceed

2. Choose recovery path:

   Option A: Fix and Re-verify
   - Identify specific tasks to re-execute
   - Re-run only those tasks
   - Re-verify affected criteria

   Option B: Re-plan Phase
   - Analyze why plan led to failure
   - Create revised plan
   - Re-execute entire phase

   Option C: Accept with Documentation
   - Document limitation in STATE.md
   - Record trade-off rationale
   - Proceed with known issue

   Option D: Abort
   - If critical failure unfixable
   - Document in BLOCKED.md
   - Escalate to project owner

3. Update ORCHESTRATION.md with decision
```

### 5.4 Prevention

- **Define clear UAT criteria** during planning
- **Check in frequently** during execution
- **Verify incrementally** – Don't wait until phase end
- **Build quality into process** – Not just at gates

---

## 6. Timeout

### 6.1 Detection

**Symptoms:**
- Iteration runs longer than expected
- No progress for extended period
- API rate limits hit
- User patience exhausted

### 6.2 Immediate Actions

1. **Force stop** if possible
2. **Record progress** – What was completed before timeout?
3. **Identify cause** – Task too large? External delays?

### 6.3 Recovery Procedure

```
1. If partial progress made:
   - Extract completed work
   - Create handoff from partial results
   - Resume from checkpoint

2. If no progress:
   - Analyze why task is too large
   - Break into smaller subtasks
   - Re-plan with smaller scope

3. If external cause (API limits, etc.):
   - Wait for limit reset
   - Reduce parallel calls
   - Use caching where possible
```

### 6.4 Prevention

- **Set realistic timeouts** per task type
- **Break large tasks** into smaller iterations
- **Monitor progress** – Check-ins at regular intervals
- **Build in buffers** – Tasks take longer than estimated

---

## 7. Recovery Decision Tree

```
Failure Detected
       │
       ▼
Is it retryable?
  ├─ Yes ─► Retry with adjustments
  │           │
  │           ▼
  │         Retry successful?
  │           ├─ Yes ─► Continue
  │           └─ No ──► Re-plan task
  │
  └─ No ──► Is it fixable?
              ├─ Yes ─► Fix and validate
              │           │
              │           ▼
              │         Fix successful?
              │           ├─ Yes ─► Continue
              │           └─ No ──► Escalate
              │
              └─ No ──► Can proceed without?
                          ├─ Yes ─► Accept with documentation
                          └─ No ──► BLOCKED
```

---

## 8. BLOCKED.md Template

When a failure cannot be resolved, create `phases/phase-N/BLOCKED.md`:

```markdown
# Blocked: Phase N - [Phase Name]

**Status:** BLOCKED
**Date:** YYYY-MM-DD
**Blocking Task:** [task_id] - [task_name]

## Failure Description

[What happened]

## Attempted Recovery

1. [Recovery attempt 1] - Result: [outcome]
2. [Recovery attempt 2] - Result: [outcome]

## Root Cause

[Why the failure occurred]

## Unblock Requirements

- [ ] [What needs to happen to unblock]
- [ ] [Resource or decision needed]

## Impact

[What's affected by this block]

## Owner

[Who can unblock this]
```

---

## Completion Status

**Task 3.4:** Error Recovery Playbook - `ERROR_PLAYBOOK_COMPLETE`

**Failure Types Documented:** 6
**Each includes:** Detection, Actions, Recovery, Prevention
**Decision Tree:** Included
**Template:** BLOCKED.md provided

