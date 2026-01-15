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

6. Generate `data-model.md` with:
   - Entities extracted from spec
   - TypeScript interfaces / Python types
   - Relationships and constraints
   - State machines (if applicable)

7. Fill `plan.md` template with:
   - Summary (from spec)
   - Technical Context (from codebase analysis)
   - Constitution Check (from constitution.md)
   - Project Structure (files to create/modify)

8. Generate `quickstart.md` with:
   - Step-by-step testing procedures
   - Expected outputs for each test
   - Troubleshooting guidance

9. Update feature `CLAUDE.md` with concrete file paths:
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
- **yes** / **proceed** - Generate tasks
- **edit** - Modify plan before task generation
- **stop** - End workflow here (artifacts saved)
```

Wait for user response before proceeding.

---

## Phase 4: Task Generation (Subagent)

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

   **⚠️ CRITICAL**: Every task MUST start with `- [ ]` checkbox - this is required for progress tracking!

4. Validate all tasks have checkboxes:
   - Scan generated tasks.md
   - Ensure EVERY task line starts with `- [ ]`
   - Fix any missing checkboxes before saving

5. Calculate complexity:
   - Count total tasks across all phases
   - Estimate complexity: Low (< 10 tasks), Medium (10-25 tasks), High (> 25 tasks)

**Output**: Report task count per phase, complexity level, MVP scope.
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
| data-model.md | Created | specs/[###-feature]/data-model.md | Entity definitions |
| plan.md | Created | specs/[###-feature]/plan.md | Technical implementation plan |
| quickstart.md | Created | specs/[###-feature]/quickstart.md | Testing procedures |
| tasks.md | Created | specs/[###-feature]/tasks.md | Task list with progress tracking |

### Implementation Ready

**Total Tasks**: [N] tasks across [N] phases
**MVP Scope**: Phase 3 (User Story 1) - [N] tasks
**Complexity**: [Low/Medium/High]

### Next Steps

1. **Start implementation with multi-agent orchestration** (recommended):
   ```
   /build specs/[###-feature]/
   ```
   Uses fresh context windows per task batch for optimal implementation.

2. **Or manual phase-by-phase implementation**:
   ```
   /speckit.implement phase:1
   /speckit.implement phase:2
   # etc.
   ```

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
