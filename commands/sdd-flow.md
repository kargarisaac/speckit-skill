---
description: Unified spec-driven development flow with codebase exploration, incremental interview, and artifact generation.
handoffs:
  - label: Implement Feature
    agent: speckit.implement
    prompt: Start implementation from Phase 1
    send: true
---

## User Input

```text
$ARGUMENTS
```

You **MUST** have a feature description in $ARGUMENTS. If empty, ask the user to provide one.

## Overview

This command provides a single-command, interview-driven workflow that combines codebase exploration, incremental specification building, and artifact generation. It replaces the manual sequence of `sdd-define → speckit.specify → speckit.clarify → speckit.plan → speckit.tasks`.

**Key Features:**
- Auto-explores codebase with parallel subagents
- Incremental interview (1-2 questions at a time, updates spec after each answer)
- Checkpoint breaks for user review
- Generates implementation prompt for handoff to another agent
- Uses subagents for heavy tasks to avoid context compaction

**Reference**: [GitHub Spec-Kit](https://github.com/github/spec-kit)

---

## Phase 1: Initialization (Automatic)

### Step 1.1: Parse Feature Request

Extract from user input:
- **Feature description**: The full description from $ARGUMENTS
- **Feature name**: Short descriptive name (2-4 words) for the branch
- **Affected areas**: UI, backend, database, etc. (inferred from description)

### Step 1.2: Codebase Exploration (Launch 4 Subagents in Parallel)

Launch **all 4 subagents simultaneously in a single message** to gather codebase context:

**Agent 1 - Tech Stack Discovery** (Explore type):
```
Explore the codebase to identify:
- Frontend framework and version (check package.json, tsconfig.json)
- Backend framework (check pyproject.toml, requirements.txt, or package.json)
- Database technology
- Key libraries used (charting, state management, UI components)
- TypeScript/JavaScript configuration
Report findings as bullet points.
```

**Agent 2 - Architecture Patterns** (Explore type):
```
Explore the codebase to identify:
- Project structure and organization
- Component patterns (how components are structured)
- State management approach
- API patterns (REST, GraphQL, etc.)
- Styling approach (CSS modules, Tailwind, CSS-in-JS, CSS variables)
Report findings as bullet points.
```

**Agent 3 - Related Code Discovery** (Explore type):
```
Based on the feature "{feature description}", find:
- Existing files that would be modified or extended
- Similar existing features to use as reference
- Relevant types, interfaces, or models
- API endpoints that might be involved
- Test patterns used in the project
Report file paths and brief descriptions.
```

**Agent 4 - Conventions & Standards** (Explore type):
```
Explore the codebase to identify:
- Naming conventions (files, components, functions)
- Code style (check .eslintrc, .prettierrc, biome.json)
- Testing framework and patterns
- Documentation standards
- Any CLAUDE.md or CONTRIBUTING.md guidelines
Report findings as bullet points.
```

### Step 1.3: Create Feature Branch

After subagents complete:

1. **Determine next feature number**:
   ```bash
   git fetch --all --prune
   ```
   - Check remote branches: `git ls-remote --heads origin | grep -oE '[0-9]{3}-' | grep -oE '[0-9]+'`
   - Check local branches: `git branch | grep -oE '[0-9]{3}-' | grep -oE '[0-9]+'`
   - Check specs directories: `ls specs/ 2>/dev/null | grep -oE '^[0-9]{3}-' | grep -oE '[0-9]+'`
   - Use highest N+1 (zero-padded to 3 digits)

2. **Generate short name** (2-4 words): Extract meaningful keywords from feature description

3. **Run create script**:
   ```bash
   ${CLAUDE_PLUGIN_ROOT}/scripts/create-new-feature.sh --json --number N --short-name "short-name" "$ARGUMENTS"
   ```
   Parse JSON output for: BRANCH_NAME, SPEC_FILE, FEATURE_NUM

### Step 1.4: Load Context

- Load `${CLAUDE_PLUGIN_ROOT}/templates/spec-template.md`
- Load `${CLAUDE_PLUGIN_ROOT}/templates/feature-claude-template.md`
- Load `.specify/memory/constitution.md`

### Step 1.5: Synthesize & Report to User

Present findings before starting interview:

```markdown
## Codebase Analysis Complete

### Tech Stack
- Frontend: [framework] + [language] + [key libraries]
- Backend: [framework] + [language]
- Database: [technology]
- Testing: [framework]

### Related Code Found
- Similar feature: `path/to/similar/feature`
- Relevant files: `path/to/file1.tsx`, `path/to/file2.py`

### Conventions
- Naming: [patterns found]
- Testing: [patterns found]

### Feature Branch Created
- Branch: `###-feature-name`
- Spec: `specs/###-feature-name/spec.md`

Ready to start interview...
```

---

## Phase 2: In-Depth Interview (Interactive)

### Core Instruction

**Interview me in detail using the `AskUserQuestion` tool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. Make sure the questions are NOT obvious.**

**Be very in-depth and continue interviewing me continually until it's complete, then write the spec to the file.**

### How to Use AskUserQuestion Tool

You **MUST** use the `AskUserQuestion` tool for ALL questions in this phase. This tool provides a structured way to ask questions with options.

**Tool Format:**
```
AskUserQuestion with parameters:
- questions: Array of 1-4 questions (ask 1-2 at a time for depth)
- Each question has:
  - question: The full question text
  - header: Short label (max 12 chars) like "Auth method", "State mgmt"
  - options: 2-4 choices with label and description
  - multiSelect: true/false (usually false)
```

### Question Categories (Interview Deeply About All)

Interview about **ALL** of these areas, not just technical ones:

| Category | What to Ask About | Example Questions |
|----------|-------------------|-------------------|
| **Technical Implementation** | Architecture, patterns, data flow, APIs | "How should data flow between components?" |
| **UI & UX** | User interactions, visual design, feedback, accessibility | "What happens when user clicks X? What feedback do they see?" |
| **Concerns & Risks** | What could go wrong, security, performance, edge cases | "What if the API fails? What if data is stale?" |
| **Tradeoffs** | Competing priorities, complexity vs simplicity | "Do you prefer faster development or more flexibility?" |
| **User Scenarios** | Who uses this, when, why, what's their goal | "Walk me through the user's journey step by step" |
| **Data & State** | What data is needed, where it lives, how it changes | "What fields does this entity need? Who can modify it?" |
| **Integration** | How it connects to existing features | "How does this interact with the existing portfolio view?" |
| **Success Criteria** | How to measure success, what "done" looks like | "How will you know this feature is working correctly?" |

### What Makes Questions NON-Obvious

**AVOID obvious questions like:**
- "What should the button say?" (can infer from feature description)
- "Should we use TypeScript?" (already in constitution.md)
- "What database should we use?" (already PostgreSQL per constitution)

**ASK non-obvious questions like:**
- "The codebase has two state patterns (Context in Dashboard, local state in Composer). Which fits this feature's data sharing needs?"
- "I see the portfolio can have 50k+ holdings. Should filtering happen client-side with virtualization or server-side with pagination?"
- "The existing alerts use toasts. But this feature might need persistent notifications. Should we add a notification center or extend toasts?"
- "What happens if the user is mid-action and their session expires? Save draft? Force re-auth? Lose work?"

### Interview Loop

**Continue asking questions until the user says "done" or you've covered all ambiguities.**

There is no fixed limit - be thorough. A complex feature might need 10+ questions. A simple one might need 3.

1. **Use AskUserQuestion tool** with 1-2 questions at a time:
   - Group related questions together (max 2 per call)
   - Wait for response before next questions
   - Each question should have 2-4 options

2. **After each answer**:
   - Update spec.md in memory with the decision
   - Show brief diff of what changed
   - Save to disk immediately (atomic updates)
   - Generate next question(s) based on new information

3. **Dig deeper when needed**:
   - If an answer reveals new complexity, ask follow-up questions
   - If the user seems uncertain, offer to explore tradeoffs
   - Don't move on until the decision is clear

### Spec Update Rules

After each answer, update the appropriate spec section:

| Answer Type | Spec Section Updated |
|-------------|---------------------|
| Scope decision | User Stories (add/remove story), Requirements (update FR) |
| User role decision | User Stories (update actor), Edge Cases |
| Integration approach | Requirements (add FR), Success Criteria (update SC) |
| UI/UX decision | User Stories (update acceptance), Edge Cases |
| Performance choice | Success Criteria (update metrics) |
| Data model choice | Key Entities, Requirements |
| Concern/risk | Edge Cases, Requirements (add defensive FR) |
| Tradeoff decision | Assumptions section, Requirements |

**Show brief diff after each update:**

```
Spec updated: Added FR-003 (server-side filtering), SC-004 (500ms response target)
```

### Termination Conditions

Stop interviewing when:
- User explicitly says "done", "proceed", "that's enough", or "let's move on"
- All major categories have been covered (technical, UI/UX, concerns, tradeoffs)
- No more [NEEDS CLARIFICATION] markers remain
- You've asked about all ambiguous areas from codebase analysis

**Do NOT stop just because you've asked a few questions. Be thorough.**

---

## CHECKPOINT 1: Spec Review

Present spec summary and wait for user approval:

```markdown
---
## CHECKPOINT: Spec Review

### spec.md Summary

**User Stories**: [N] defined (P1, P2, P3)
**Requirements**: [N] functional requirements
**Success Criteria**: [N] measurable outcomes
**Clarifications Made**: [N] questions resolved

<details>
<summary>Full spec.md content (click to expand)</summary>

[Full spec content here]

</details>

---

**Options**:
- **yes** / **proceed** - Continue to plan generation
- **edit** - Make changes before proceeding (describe what to change)
- **regenerate** - Start interview over with new approach
```

Wait for user response before proceeding.

---

## Phase 3: Plan Generation (Subagent)

After user approves spec at Checkpoint 1, launch **ONE Plan subagent** to generate artifacts:

**Plan Subagent Prompt:**
```
You are generating implementation artifacts for feature: [feature name]

**Context:**
- Branch: [###-feature-name]
- Spec location: specs/[###-feature-name]/spec.md
- Codebase tech stack: [summary from Phase 1]
- Codebase patterns: [summary from Phase 1]

**Tasks:**
1. Run `${CLAUDE_PLUGIN_ROOT}/scripts/setup-plan.sh --json` and parse output
2. Load the spec at FEATURE_SPEC path
3. Load `.specify/memory/constitution.md` for project constraints

4. Generate `research.md` with:
   - Technical decisions made (from interview)
   - Best practices for the tech stack
   - Integration approaches
   Format: Decision → Rationale → Alternatives Considered

5. Generate `decisions.md` with ADRs:
   - Convert each major decision from interview/research into an ADR
   - Each ADR should include: Context, Options Considered, Decision, Consequences
   - Number ADRs sequentially (ADR-001, ADR-002, etc.)
   - This captures the "why" for future sessions and contributors

6. Generate `prompts/` folder with reusable prompts:
   - Create `prompts/implement.md` - Implementation prompt
   - Create `prompts/test.md` - Test commands and debug prompts
   - Create `prompts/review.md` - Code review prompts
   - These become reusable copy-paste prompts during development

8. Generate `data-model.md` with:
   - Entities extracted from spec
   - TypeScript interfaces / Python types
   - Relationships and constraints
   - State machines (if applicable)

9. Fill `plan.md` template with:
   - Summary (from spec)
   - Technical Context (from codebase analysis)
   - Constitution Check (from constitution.md)
   - Project Structure (files to create/modify)

10. Generate `quickstart.md` with:
    - Step-by-step testing procedures
    - Expected outputs for each test
    - Troubleshooting guidance

11. Update feature `CLAUDE.md` with concrete file paths:
    - Update "Files to Modify" with actual files to change
    - Update "Files to Create" with new files to add
    - Update "Reference Implementations" with similar code patterns found
    - Add feature-specific code patterns from decisions.md
    - Update "Don't" rules based on decisions made

**Output**: Report paths to all generated files (including decisions.md and updated CLAUDE.md) and a summary of decisions made.
```

---

## CHECKPOINT 2: Plan Review

Present plan summary and wait for user approval:

```markdown
---
## CHECKPOINT: Plan Review

### Generated Artifacts
- research.md: [N] decisions documented
- decisions.md: [N] ADRs captured (the "why" behind decisions)
- prompts/: Reusable prompts folder (implement.md, test.md, review.md)
- data-model.md: [N] entities defined
- plan.md: Complete technical plan
- quickstart.md: Testing procedures
- CLAUDE.md: Updated with concrete file paths

### Architecture Summary
- Project Structure: [summary]
- New Files: [N] files to create
- Modified Files: [N] files to update

<details>
<summary>plan.md content (click to expand)</summary>

[plan.md content here]

</details>

---

**Options**:
- **yes** / **proceed** - Generate tasks and implementation prompt
- **edit** - Modify plan before task generation
- **stop** - End workflow here (artifacts saved)
```

Wait for user response before proceeding.

---

## Phase 4: Task & Prompt Generation (Subagent)

After user approves plan at Checkpoint 2, launch **ONE Plan subagent** to generate tasks:

**Task Generation Subagent Prompt:**
```
You are generating tasks for feature: [feature name]

**Context:**
- Branch: [###-feature-name]
- Feature directory: specs/[###-feature-name]/

**Tasks:**
1. Run `${CLAUDE_PLUGIN_ROOT}/scripts/check-prerequisites.sh --json` and parse output
2. Load all design documents:
   - spec.md (required - user stories)
   - plan.md (required - tech stack)
   - data-model.md (entities)
   - research.md (decisions)

3. Generate `tasks.md` using `${CLAUDE_PLUGIN_ROOT}/templates/tasks-template.md` structure:
   - Phase 1: Setup tasks
   - Phase 2: Foundational (blocking) tasks
   - Phase 3+: One phase per user story (P1, P2, P3...)
   - Final Phase: Polish tasks

   Task format: `- [ ] [TaskID] [P?] [Story] Description with file path`
   Mark parallelizable tasks with [P]

4. Calculate complexity and Ralph Loop parameters:
   - Count total tasks across all phases
   - Estimate complexity: Low (< 10 tasks), Medium (10-25 tasks), High (> 25 tasks)
   - Recommend max-iterations: Low=30, Medium=50, High=80

5. Generate `IMPLEMENTATION_PROMPT.md` (see template below):
   - Replace [ITERATIONS] with calculated max-iterations
   - Replace [Feature Name] with actual feature name
   - Replace [###-feature-name] with actual branch name
   - List all success criteria from spec.md in "Completion Criteria" section

6. Generate `REVIEW_PROMPT.md` (see template below)

**Output**: Report task count per phase, complexity level, recommended max-iterations, MVP scope.
```

### IMPLEMENTATION_PROMPT.md Template

Generate this file at `specs/[###-feature]/IMPLEMENTATION_PROMPT.md`:

```markdown
# Implementation Prompt: [Feature Name]

**Generated**: [DATE]
**Branch**: [###-feature-name]
**Estimated Complexity**: [Low/Medium/High based on task count]

Note: Use subagents where you can to keep the orchestrator's context free. Also, use parallel subagents when possible.

## 🔄 Ralph Loop Usage

**This prompt is optimized for Ralph Loop autonomous execution.**

**Command:**
```bash
/ralph-loop:ralph-loop "$(cat specs/[###-feature]/IMPLEMENTATION_PROMPT.md)" --max-iterations [ITERATIONS] --completion-promise "FEATURE_COMPLETE"
```

**Max Iterations Guidance:**
- **Low Complexity** (< 10 tasks): --max-iterations 30
- **Medium Complexity** (10-25 tasks): --max-iterations 50
- **High Complexity** (> 25 tasks): --max-iterations 80

**Completion Signal:**
When ALL completion criteria pass (see end of prompt), output:
```
<promise>FEATURE_COMPLETE</promise>
```

**Do NOT output this promise until every criterion objectively passes.**

## ⚠️ CRITICAL: Progress Tracking Rules

**After EACH task, you MUST immediately:**

1. **Mark the task [X]** in tasks.md (change `- [ ]` to `- [x]`)
2. **Update the Progress section** at the top of tasks.md with what you just completed
3. **Then** proceed to the next task

**The rule**: `Complete task → Mark [X] → Update Progress → Next task`

**Why?** Context can reset at any time. If you don't update progress continuously, all work is lost to the next session. This is non-negotiable.

**Never**: Complete multiple tasks then mark them all done later. That's how work gets lost.

## Quick Context

[2-3 sentence summary of what this feature does and why]

## Codebase Orientation

### Files to Modify
- `path/to/file1.tsx` - [What changes needed]
- `path/to/file2.py` - [What changes needed]

### Files to Create
- `path/to/new/file1.tsx` - [Purpose]
- `path/to/new/file2.py` - [Purpose]

### Reference Implementations
- `path/to/similar/feature/` - Follow this pattern for [component/service]

## Tech Stack (Already Determined)

- **Frontend**: [from plan.md]
- **Backend**: [from plan.md]
- **Database**: [from plan.md]
- **Key Libraries**: [from plan.md]

## Implementation Order

1. **Phase 1: Setup** - [X tasks]
2. **Phase 2: Foundational** - [X tasks]
3. **Phase 3: User Story 1 (MVP)** - [X tasks]
4. **Phase 4: User Story 2** - [X tasks]
...

## Critical Constraints

[From constitution.md]

- [ ] Must use PostgreSQL only (no SQLite)
- [ ] Must follow AG-UI component patterns
- [ ] Must include backend integration tests
- [ ] [Other project-specific constraints]

## Ready-to-Run Commands

```bash
# For manual phase-by-phase implementation
/speckit.implement phase:1

# For Ralph Loop autonomous implementation (recommended)
/ralph-loop:ralph-loop "$(cat specs/[###-feature]/IMPLEMENTATION_PROMPT.md)" --max-iterations [30-80] --completion-promise "FEATURE_COMPLETE"
```

## Self-Verification Loop (REQUIRED)

**After completing EACH phase, automatically verify before proceeding:**

1. **Mark phase tasks [X]** in tasks.md
2. **Run backend tests** (if backend changes):
   ```bash
   uv run pytest tests/ -v
   ```
   - If ANY test fails: debug → fix → re-run → repeat until all pass

3. **Run UI tests** (if frontend changes):
   ```bash
   # Start backend if not running
   ./local_run.sh

   # Use chrome-in-claude MCP tool to:
   # - Open the UI in browser
   # - Navigate to the new/modified feature
   # - Interact with all UI elements
   # - Verify: no console errors, correct rendering, proper functionality
   # - Test edge cases from spec.md
   ```
   - If ANY issue found: fix → re-verify → repeat

4. **Run integration tests** (if applicable):
   ```bash
   # Test API endpoints manually or with integration test suite
   # Verify database changes (check PostgreSQL directly if needed)
   # Test full user flows from spec.md user stories
   ```

5. **Only proceed to next phase** when current phase is 100% verified

**Never skip verification steps. Real testing > assumptions.**

## Completion Criteria

**ALL of the following MUST be true before outputting completion promise:**

### 1. Tasks Complete
- [ ] All tasks in `specs/[###-feature]/tasks.md` marked [X]
- [ ] Progress section updated with final status

### 2. Backend Tests Pass (if backend changes)
- [ ] `uv run pytest tests/` exits with 0 (all tests pass)
- [ ] No pytest warnings or errors
- [ ] New backend code has test coverage (aim for 80%+)

### 3. UI Verification Pass (if frontend changes)
**Using chrome-in-claude MCP tool:**
- [ ] UI renders without errors (check browser console)
- [ ] All interactive elements work (buttons, forms, navigation)
- [ ] All user stories from spec.md can be completed in the UI
- [ ] Edge cases from spec.md are handled correctly
- [ ] Responsive design works (if applicable)
- [ ] Accessibility: keyboard navigation works, ARIA labels present

**Command:**
```bash
# Ensure backend is running
./local_run.sh

# Then use MCP tool to open and test:
# mcp-cli call claude-in-chrome/navigate '{"url": "http://localhost:3000/your-feature"}'
# mcp-cli call claude-in-chrome/read_page '{}'
# mcp-cli call claude-in-chrome/computer '{"action": "click", ...}'
# etc.
```

### 4. Integration Tests Pass
- [ ] Full user flows work end-to-end (frontend → backend → database → response)
- [ ] API endpoints return expected data
- [ ] Database state is correct after operations
- [ ] No unhandled errors in backend logs

### 5. Success Criteria from spec.md
- [ ] [List each SC-XXX from spec.md here]
- [ ] All functional requirements (FR-XXX) are implemented
- [ ] All edge cases are handled

### 6. Code Quality
- [ ] No TypeScript errors (if frontend): `npx tsc --noEmit`
- [ ] No linting errors (if applicable)
- [ ] No console.log or debug code left behind
- [ ] All imports used, no dead code

### 7. Documentation
- [ ] README updated (if needed)
- [ ] API documentation updated (if backend changes)
- [ ] Comments added for non-obvious logic

---

## When All Criteria Pass

**Output the completion promise:**

```
<promise>FEATURE_COMPLETE</promise>
```

**If ANY criterion fails, do NOT output the promise. Instead:**
1. Identify what failed
2. Fix the issue
3. Re-run verification
4. Repeat until everything passes

---

**Full Artifacts**: See `specs/[###-feature]/` for complete spec.md, plan.md, tasks.md
```

### REVIEW_PROMPT.md Template

Generate this file at `specs/[###-feature]/REVIEW_PROMPT.md`:

```markdown
# Review Prompt: [Feature Name]

**Generated**: [DATE]
**Branch**: [###-feature-name]
**Purpose**: Validate implementation against specification

Note: Use subagents to review different aspects in parallel (UI review, backend review, test coverage review, etc.)

## Your Role

You are a critical code reviewer. Your job is to find gaps, issues, discrepancies, and problems between the implementation and the specification. Be thorough and skeptical.

## Specification Reference

**Feature Spec**: `specs/[###-feature]/spec.md`
**Implementation Plan**: `specs/[###-feature]/plan.md`
**Data Model**: `specs/[###-feature]/data-model.md`
**Task List**: `specs/[###-feature]/tasks.md`

## Review Checklist

### 1. Requirement Coverage

For EACH functional requirement in spec.md:
- [ ] FR-001: Is it implemented? Where? Does it match the spec exactly?
- [ ] FR-002: ...
- [ ] [Continue for all FRs]

**Report**: List any FRs that are missing, partially implemented, or deviate from spec.

### 2. User Story Validation

For EACH user story in spec.md:
- [ ] US1: Do all acceptance scenarios pass? Test each Given/When/Then.
- [ ] US2: ...
- [ ] [Continue for all user stories]

**Report**: List any acceptance scenarios that fail or behave unexpectedly.

### 3. Data Model Conformance

Compare implementation to data-model.md:
- [ ] All entities exist with correct fields
- [ ] All relationships are correctly implemented
- [ ] All validation rules are enforced
- [ ] All state transitions work correctly

**Report**: List any entity/field mismatches, missing validations, or broken relationships.

### 4. Edge Cases

For EACH edge case in spec.md:
- [ ] Is it handled? How?
- [ ] Does the handling match the spec?

**Report**: List any unhandled or incorrectly handled edge cases.

### 5. Success Criteria

For EACH success criterion in spec.md:
- [ ] SC-001: Is the metric met? How was it verified?
- [ ] SC-002: ...
- [ ] [Continue for all SCs]

**Report**: List any success criteria that are not met or cannot be verified.

### 6. Constitution Compliance

Check against `.specify/memory/constitution.md`:
- [ ] Type safety enforced (TypeScript strict, Python type hints)
- [ ] PostgreSQL only (no SQLite, CSV, in-memory)
- [ ] Tests exist (pytest for backend)
- [ ] React + AG-UI patterns followed (if UI)
- [ ] Performance targets met (<500ms API, <3s dashboard load)
- [ ] Accessibility requirements (WCAG 2.1 AA)

**Report**: List any constitution violations.

### 7. Code Quality Issues

Look for:
- [ ] Dead code or unused imports
- [ ] Missing error handling
- [ ] Security vulnerabilities (injection, XSS, etc.)
- [ ] Performance issues (N+1 queries, unnecessary re-renders)
- [ ] Missing or inadequate tests
- [ ] Inconsistent naming or patterns

**Report**: List all code quality issues found.

### 8. Discrepancies

Compare implementation to plan.md:
- [ ] Are all files in "Files to Create" actually created?
- [ ] Are all files in "Files to Modify" actually modified?
- [ ] Does the project structure match the plan?

**Report**: List any files missing, extra files not in plan, or structural differences.

## Output Format

After review, produce a report with:

```markdown
## Review Summary: [Feature Name]

**Reviewer**: [Agent/Human]
**Date**: [DATE]
**Overall Status**: [PASS / PASS WITH ISSUES / FAIL]

### Critical Issues (Must Fix)
1. [Issue description, file path, spec reference]
2. ...

### Major Issues (Should Fix)
1. [Issue description, file path, spec reference]
2. ...

### Minor Issues (Nice to Fix)
1. [Issue description, file path, spec reference]
2. ...

### Gaps (Missing from Spec)
1. [What's missing, spec reference]
2. ...

### Discrepancies (Different from Spec)
1. [What differs, expected vs actual, spec reference]
2. ...

### Positive Observations
1. [What was done well]
2. ...

### Recommendations
1. [Suggested improvement]
2. ...
```

## Verification Commands

Run these to assist your review:

```bash
# Run all tests
uv run pytest tests/ -v

# Type check
npx tsc --noEmit  # Frontend
# or use pyright/mypy for Python

# Lint check
npm run lint  # Frontend
# or use ruff for Python

# Check for TODOs/FIXMEs
grep -r "TODO\|FIXME\|XXX\|HACK" src/
```

## Review Approach

1. **Read the spec first** - Understand what was supposed to be built
2. **Read the implementation** - Understand what was actually built
3. **Compare systematically** - Go through each checklist item
4. **Be skeptical** - Assume there are bugs until proven otherwise
5. **Be specific** - Quote file paths, line numbers, and spec references
6. **Prioritize** - Critical > Major > Minor
```

---

## Phase 5: Completion Report

Present final summary:

```markdown
---
## Flow Complete

### Artifacts Created
| File | Status | Location | Purpose |
|------|--------|----------|---------|
| spec.md | Created | specs/[###-feature]/spec.md | Requirements & user stories |
| CLAUDE.md | Created | specs/[###-feature]/CLAUDE.md | Feature-specific Claude instructions |
| research.md | Created | specs/[###-feature]/research.md | Technical research findings |
| decisions.md | Created | specs/[###-feature]/decisions.md | ADRs - the "why" behind decisions |
| prompts/ | Created | specs/[###-feature]/prompts/ | Reusable prompts folder (implement.md, test.md, review.md) |
| data-model.md | Created | specs/[###-feature]/data-model.md | Entity definitions |
| plan.md | Created | specs/[###-feature]/plan.md | Technical implementation plan |
| quickstart.md | Created | specs/[###-feature]/quickstart.md | Testing procedures |
| tasks.md | Created | specs/[###-feature]/tasks.md | Task list with progress tracking |
| IMPLEMENTATION_PROMPT.md | Created | specs/[###-feature]/IMPLEMENTATION_PROMPT.md | Implementation handoff |
| REVIEW_PROMPT.md | Created | specs/[###-feature]/REVIEW_PROMPT.md | Review checklist |

### Implementation Ready

**Total Tasks**: [N] tasks across [N] phases
**MVP Scope**: Phase 3 (User Story 1) - [N] tasks
**Complexity**: [Low/Medium/High]
**Recommended Max Iterations**: [30/50/80]

### Next Steps

1. **Review implementation prompt**:
   ```
   Read specs/[###-feature]/IMPLEMENTATION_PROMPT.md
   ```

2. **Start implementation with Ralph Loop** (recommended):
   ```bash
   /ralph-loop:ralph-loop "$(cat specs/[###-feature]/IMPLEMENTATION_PROMPT.md)" --max-iterations [30-80] --completion-promise "FEATURE_COMPLETE"
   ```
   Ralph will autonomously implement, test, and verify until completion.

3. **Or manual phase-by-phase implementation**:
   ```
   /speckit.implement phase:1
   /speckit.implement phase:2
   # etc.
   ```

4. **After implementation, run review** (new agent session):
   ```
   Read specs/[###-feature]/REVIEW_PROMPT.md
   ```
   This prompt will guide a reviewer to find gaps, issues, and discrepancies.

---

**Branch**: `[###-feature-name]`
**All artifacts saved in**: `specs/[###-feature]/`
```

---

## Appendix A: AskUserQuestion Examples

### Example 1: Technical + Integration Questions

```
AskUserQuestion({
  questions: [
    {
      question: "Your API uses REST patterns (src/nazmi/api/*.py). This feature needs real-time portfolio updates. How should updates be delivered?",
      header: "Real-time",
      options: [
        { label: "WebSocket (Recommended)", description: "Add WS endpoint alongside REST, real-time push, matches FastAPI patterns" },
        { label: "Server-Sent Events", description: "Simpler one-way updates, wider browser support, less infrastructure" },
        { label: "Polling", description: "No new infrastructure needed, higher latency, more server load" }
      ],
      multiSelect: false
    },
    {
      question: "Found React Context in web/src/contexts/ and local state in components. How should this feature manage state?",
      header: "State mgmt",
      options: [
        { label: "React Context (Recommended)", description: "Match existing pattern, add new context provider" },
        { label: "Local component state", description: "Simpler, no shared state, may need prop drilling" },
        { label: "URL state", description: "Shareable links, bookmarkable, limited to simple data" }
      ],
      multiSelect: false
    }
  ]
})
```

### Example 2: UI/UX + Concerns Questions

```
AskUserQuestion({
  questions: [
    {
      question: "The existing alerts use toast notifications. This feature might need persistent notifications. What approach should we take?",
      header: "Notifications",
      options: [
        { label: "Extend toasts", description: "Add 'sticky' toast option, simpler, consistent with current UX" },
        { label: "Notification center", description: "New dropdown with history, more complex, better for many notifications" },
        { label: "In-page alerts", description: "Show alerts inline in the relevant component, no overlay" }
      ],
      multiSelect: false
    },
    {
      question: "What happens if the user is mid-action and their session expires?",
      header: "Session edge",
      options: [
        { label: "Auto-save draft", description: "Save work to localStorage, restore after re-auth" },
        { label: "Force re-auth", description: "Redirect to login, lose unsaved work, simpler" },
        { label: "Extend session", description: "Silently refresh token if possible, only prompt if truly expired" }
      ],
      multiSelect: false
    }
  ]
})
```

### Example 3: Tradeoffs + Success Criteria

```
AskUserQuestion({
  questions: [
    {
      question: "Portfolio can have 50k+ holdings. Filtering tradeoff: faster initial load vs. more responsive filtering?",
      header: "Perf tradeoff",
      options: [
        { label: "Server-side (Recommended)", description: "Pagination, <500ms response, requires API changes" },
        { label: "Client-side + virtualization", description: "Load all data once, instant filtering, heavier initial load" },
        { label: "Hybrid", description: "Server filters, client sorts/searches, more complex" }
      ],
      multiSelect: false
    }
  ]
})
```

### Example 4: Deep Dive on User Scenario

```
AskUserQuestion({
  questions: [
    {
      question: "Walk me through: when a user wants to add a new holding, what's their starting point?",
      header: "Entry point",
      options: [
        { label: "Portfolio page button", description: "Dedicated 'Add Holding' button on portfolio view" },
        { label: "Quick action menu", description: "Global '+' button accessible from anywhere" },
        { label: "Search result action", description: "Find stock first, then 'Add to portfolio' from search" },
        { label: "Multiple entry points", description: "All of the above, user chooses their workflow" }
      ],
      multiSelect: false
    }
  ]
})
```

---

## Appendix B: Script Integration Reference

| Phase | Script | Purpose |
|-------|--------|---------|
| 1 | `create-new-feature.sh --json` | Create branch and spec directory |
| 3 | `setup-plan.sh --json` | Get paths, copy plan template |
| 4 | `check-prerequisites.sh --json` | Validate artifacts before tasks |

---

## Behavior Rules

- **If $ARGUMENTS is empty**: Ask user for feature description before proceeding
- **If codebase exploration fails**: Report findings but continue with available context
- **ALWAYS use `AskUserQuestion` tool** for Phase 2 interview - never use plain markdown questions
- **Be thorough in interview**: No fixed question limit - continue until user says "done" or all ambiguities covered
- **Cover all categories**: Technical, UI/UX, concerns, tradeoffs, user scenarios, data, integration, success criteria
- **If user says "done" during interview**: Proceed to Checkpoint 1 with current spec
- **If user rejects at checkpoint**: Allow edits or regeneration
- **If subagent fails**: Report error and allow retry or manual intervention
- **Always use subagents** for codebase exploration (Phase 1) and artifact generation (Phase 3, 4) to avoid context compaction
