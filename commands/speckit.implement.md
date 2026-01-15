---
description: Execute implementation using multi-agent orchestration with fresh context windows per task batch
argument-hint: "<spec-folder-path> [--continue] [--phase <n>]"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** parse the user input before proceeding.

## Overview

This command implements features using a **multi-agent orchestration system**. Each task batch is handled by a fresh subagent to avoid context bloat, while the lead orchestrator tracks progress in `tasks.md`.

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    LEAD ORCHESTRATOR (You)                  │
│  Reads: spec.md, plan.md, tasks.md, decisions.md            │
│  Tracks: Progress in tasks.md, batch completion             │
│  Controls: Task grouping, parallelization, test triggers    │
└─────────────────────────────────────────────────────────────┘
              │                              │
              ▼                              ▼
┌─────────────────────────┐    ┌─────────────────────────────┐
│   IMPLEMENTER SUBAGENT  │    │      TESTER SUBAGENT        │
│   (Fresh context)       │    │      (Fresh context)        │
│                         │    │                             │
│   - Implements tasks    │    │   - E2E tests after batch   │
│   - Marks [X] in tasks  │    │   - Uses browser MCP        │
│   - Groups small tasks  │    │   - Verifies functionality  │
└─────────────────────────┘    └─────────────────────────────┘
```

---

## Step 1: Parse Arguments

Parse `$ARGUMENTS` for:

1. **Spec folder path** (required unless `--continue`):
   - Example: `specs/001-user-auth/`
   - Can be relative or absolute path

2. **Flags**:
   - `--continue`: Resume from last incomplete task (uses Progress section in tasks.md)
   - `--phase <n>`: Execute only phase N, then stop

**Error handling:**
- If no path and no `--continue` flag: Ask user for spec folder path
- If path doesn't exist: Report error and stop
- If required files missing: Report which files are missing and stop

---

## Step 2: Load and Validate Spec Files

Read all files from the spec folder and validate:

| File | Required | Purpose |
|------|----------|---------|
| `spec.md` | ✅ Yes | User stories, requirements, success criteria |
| `plan.md` | ✅ Yes | Technical implementation plan, file structure |
| `tasks.md` | ✅ Yes | Task list with phases, dependencies, progress |
| `data-model.md` | Optional | Entity definitions and relationships |
| `decisions.md` | Optional | ADRs - architecture decision records |
| `quickstart.md` | Optional | Testing procedures and commands |

**Report to user:**
```markdown
## Spec Files Loaded

| File | Status | Summary |
|------|--------|---------|
| spec.md | ✅ Found | [N] user stories, [N] requirements |
| plan.md | ✅ Found | [N] files to modify, [N] files to create |
| tasks.md | ✅ Found | [N] tasks across [N] phases |
| data-model.md | ⚠️ Not found | (optional) |
| decisions.md | ✅ Found | [N] ADRs |
| quickstart.md | ✅ Found | Test commands available |
```

---

## Step 3: Analyze Current State

Parse `tasks.md` to understand:

1. **Progress status:**
   - Read the `## Progress (Session Continuity)` section
   - Count completed tasks: `- [x]` or `- [X]`
   - Count incomplete tasks: `- [ ]`
   - Identify current phase

2. **Task structure:**
   - Extract all phases (Setup, Foundational, User Stories, Polish)
   - Identify parallel markers `[P]` for tasks that can run together
   - Note dependencies between tasks

3. **Report current state:**
```markdown
## Current Progress

**Completed**: [N]/[Total] tasks ([X]%)
**Current Phase**: [Phase Name]
**Next Tasks**: [List of next 3-5 incomplete tasks]
```

---

## Step 4: Main Implementation Loop

**CRITICAL:** This loop continues until ALL tasks in `tasks.md` are marked `[x]`.

```
WHILE incomplete tasks exist in tasks.md:

  4.1 DETERMINE NEXT BATCH
      - Read tasks.md for incomplete tasks
      - Group tasks dynamically based on:
        - Same phase or feature area
        - [P] markers indicate parallelizable tasks
        - Small tasks can be grouped (2-3 per batch)
        - Respect dependencies (don't batch dependent tasks)

  4.2 DISPATCH IMPLEMENTER SUBAGENT(S)
      - For independent [P] tasks: Launch parallel subagents
      - For dependent tasks: Launch single sequential subagent
      - Use Task tool with subagent_type="general-purpose"
      - Include IMPLEMENTER PROMPT (see below)

  4.3 WAIT FOR COMPLETION
      - Collect results from subagent(s)
      - Re-read tasks.md to verify tasks marked [x]
      - Log any errors or issues

  4.4 HANDLE FAILURES
      - If subagent failed: RETRY ONCE with fresh context
      - If retry also fails: STOP and report error to user
      - Ask user: "Retry again?", "Skip this task?", or "Stop?"

  4.5 TEST TRIGGER (after each batch)
      - Launch TESTER SUBAGENT with fresh context
      - Use TESTER PROMPT (see below)
      - Report test results (PASS/FAIL with details)

  4.6 PROGRESS CHECK
      - Re-read tasks.md
      - Report: "Completed [N]/[Total] tasks"
      - If all done: Exit loop

END WHILE
```

---

## Step 5: Implementer Subagent Prompt

When dispatching an implementer subagent, use this prompt template:

```markdown
You are implementing tasks for feature: {feature_name}

## Context
- Spec folder: {spec_path}
- Current batch: Tasks {task_ids}

## Files Reference
{Excerpts from plan.md - Files to Modify, Files to Create}

## Tasks to Implement
{List of tasks from tasks.md for this batch}

## Instructions
1. Implement each task following the plan.md architecture
2. After completing EACH task, IMMEDIATELY:
   - Mark it [x] in {spec_path}/tasks.md (change `- [ ]` to `- [x]`)
   - Update the Progress section at the top of tasks.md
3. If you encounter blockers, document them in the Progress section and continue with other tasks
4. Do NOT implement tasks outside this batch
5. Follow code patterns from decisions.md (ADRs)

## Constraints
{Relevant ADRs from decisions.md}

## What Success Looks Like
- All tasks in this batch are implemented
- All tasks marked [x] in tasks.md
- Progress section updated
- Code follows plan.md architecture
```

---

## Step 6: Tester Subagent Prompt

After each batch, launch a tester subagent:

```markdown
You are testing the implementation for feature: {feature_name}

## Context
- Spec folder: {spec_path}
- Just completed: Tasks {completed_task_ids}

## What to Test
{Relevant user stories from spec.md for the completed tasks}

## Testing Commands
{From quickstart.md if available, otherwise suggest appropriate commands}

## Instructions

### 1. Run Automated Tests (if available)
```bash
# Backend tests
pytest tests/ -v

# Frontend tests
npm test
```

### 2. For UI Features - Use Browser MCP
**Primary**: chrome-in-claude MCP tool
**Fallback**: chrome-devtools MCP tool

Steps:
a. Navigate to the relevant page
b. Test the implemented functionality
c. Check browser console for errors
d. Verify user interactions work

Example commands:
```bash
mcp-cli call chrome-in-claude/navigate '{"url": "http://localhost:3000/feature-path"}'
mcp-cli call chrome-in-claude/read_page '{}'
mcp-cli call chrome-in-claude/computer '{"action": "click", "coordinate": [x, y]}'
```

If chrome-in-claude is unavailable, try:
```bash
mcp-cli call chrome-devtools/navigate '{"url": "http://localhost:3000/feature-path"}'
```

### 3. Report Results
- **PASS**: All tests passed - provide summary
- **FAIL**: What failed, error messages, and suggestions for fix

## Success Criteria from spec.md
{Relevant success criteria excerpts}
```

---

## Step 7: Completion

When all tasks are marked `[x]` in tasks.md:

1. **Run final verification:**
   - Launch one more tester subagent for full E2E test
   - Verify all user stories from spec.md work

2. **Report completion:**
```markdown
## Implementation Complete ✅

**Feature**: {feature_name}
**Spec folder**: {spec_path}

### Summary
- Total tasks completed: [N]
- Phases completed: [list]
- Tests: [PASS/FAIL status]

### Files Modified
{List from plan.md}

### Files Created
{List from plan.md}

### Next Steps
1. Review the implementation
2. Run full test suite: `[test command]`
3. Create PR when ready
```

---

## Key Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Fresh context per subagent** | Each batch gets a new subagent to avoid context bloat |
| **tasks.md as single source of truth** | All progress tracked in tasks.md, survives context resets |
| **Dynamic batch sizing** | Lead agent decides batch size based on task complexity |
| **Test after every batch** | Catches issues early, not at the end |
| **Retry once on failure** | If subagent fails, retry with fresh context; stop if still failing |
| **Browser MCP for UI** | Use chrome-in-claude (primary) or chrome-devtools (fallback) |

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Spec folder not found | Report error, ask user to provide correct path |
| Required file missing | Report which file, suggest running `/speckit.tasks` first |
| Subagent fails | Retry once with fresh context |
| Retry also fails | Stop and ask user what to do |
| Test fails | Report failure, continue with next batch (don't block) |
| Browser MCP unavailable | Skip UI tests, report that manual testing is needed |

---

## Usage Examples

```bash
# Start implementation from scratch
/speckit.implement specs/001-user-auth/

# Resume from where you left off
/speckit.implement --continue

# Execute only phase 2
/speckit.implement specs/001-user-auth/ --phase 2

# Relative path
/speckit.implement ./specs/002-dashboard/
```

---

## Notes

- This command uses the Task tool to spawn subagents
- Each subagent has fresh context (no memory of previous batches)
- Progress is tracked in tasks.md to survive context resets
- The lead orchestrator (you) maintains overall progress awareness
- Parallel execution uses [P] markers from tasks.md
