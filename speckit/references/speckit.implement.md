---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Phase Selection

Parse `$ARGUMENTS` for phase selection:

- **No arguments**: Execute ALL phases sequentially
- **`phase:1`** or **`phase:setup`**: Execute Phase 1 (Setup) only, then STOP
- **`phase:2`** or **`phase:foundational`**: Execute Phase 2 (Foundational) only, then STOP
- **`phase:3`** or **`phase:us1`**: Execute Phase 3 (User Story 1) only, then STOP
- **`phase:4`** or **`phase:us2`**: Execute Phase 4 (User Story 2) only, then STOP
- **`phase:5`** or **`phase:us3`**: Execute Phase 5 (User Story 3) only, then STOP
- **`phase:N`**: Execute Phase N only, then STOP
- **`phase:polish`**: Execute the final Polish phase only, then STOP
- **`continue`**: Skip completed tasks (marked [X]), continue from first incomplete task

**IMPORTANT**: When a phase argument is provided:
1. Skip all tasks from previous phases (or verify they are marked [X])
2. Execute ONLY tasks in the specified phase
3. Mark completed tasks as [X] in tasks.md
4. STOP after the phase completes - do NOT proceed to next phase
5. Report: "Phase N complete. Run `/speckit.implement phase:N+1` to continue."

## Outline

1. Run `${SPECKIT_ROOT}/scripts/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
   - Scan all checklist files in the checklists/ directory
   - For each checklist, count:
     - Total items: All lines matching `- [ ]` or `- [X]` or `- [x]`
     - Completed items: Lines matching `- [X]` or `- [x]`
     - Incomplete items: Lines matching `- [ ]`
   - Create a status table:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - Calculate overall status:
     - **PASS**: All checklists have 0 incomplete items
     - **FAIL**: One or more checklists have incomplete items

   - **If any checklist is incomplete**:
     - Display the table with incomplete item counts
     - **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     - Wait for user response before continuing
     - If user says "no" or "wait" or "stop", halt execution
     - If user says "yes" or "proceed" or "continue", proceed to step 3

   - **If all checklists are complete**:
     - Display the table showing all checklists passed
     - Automatically proceed to step 3

3. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

4. **Project Setup Verification**:
   - **REQUIRED**: Create/verify ignore files based on actual project setup:

   **Detection & Creation Logic**:
   - Check if the following command succeeds to determine if the repository is a git repo (create/verify .gitignore if so):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Check if Dockerfile* exists or Docker in plan.md → create/verify .dockerignore
   - Check if .eslintrc* exists → create/verify .eslintignore
   - Check if eslint.config.* exists → ensure the config's `ignores` entries cover required patterns
   - Check if .prettierrc* exists → create/verify .prettierignore
   - Check if .npmrc or package.json exists → create/verify .npmignore (if publishing)
   - Check if terraform files (*.tf) exist → create/verify .terraformignore
   - Check if .helmignore needed (helm charts present) → create/verify .helmignore

   **If ignore file already exists**: Verify it contains essential patterns, append missing critical patterns only
   **If ignore file missing**: Create with full pattern set for detected technology

   **Common Patterns by Technology** (from plan.md tech stack):
   - **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `Makefile`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **Tool-Specific Patterns**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

6. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

7. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

8. **CRITICAL: Continuous Progress Tracking** ⚠️

   **After EACH task completion, you MUST immediately:**

   a. **Mark the task [X] in tasks.md**:
      - Change `- [ ] T00X` to `- [x] T00X`
      - Do this IMMEDIATELY after completing each task, not in batches
      - Never proceed to the next task without marking the current one done

   b. **Update the Progress section** at the top of tasks.md:
      ```markdown
      ## Progress (Session Continuity)

      ### What's Done
      - Phase 1: Complete (T001-T003)
      - T004: Created database schema  ← ADD EACH TASK AS YOU COMPLETE IT

      ### Currently In Progress
      - T005: Implementing user service  ← UPDATE THIS LIVE

      ### Next Session Should
      1. Continue with T006 if T005 is incomplete
      ```

   c. **Why this is non-negotiable**:
      - Context can reset at any time (30% usage)
      - If you don't update progress, all work is lost to the next session
      - The next session/assistant needs to know exactly where you stopped
      - This is how Ashpreet Bedi's team maintains session continuity

   d. **The rule**:
      ```
      Complete task → Mark [X] → Update Progress → THEN next task
      ```
      Never: Complete task → Complete task → Complete task → Mark all done later

9. Error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **If you must stop mid-phase**: Update Progress section with exactly where you stopped

10. Completion validation:
   - **If phase argument provided**: Verify all tasks in that phase are completed, then STOP
   - **If no phase argument**: Verify all required tasks across all phases are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

11. Phase completion reporting:
   - **If phase argument was provided**:
     - Report: "✅ Phase N complete. X tasks completed."
     - Report: "Next: Run `/speckit.implement phase:N+1` in a new session to continue."
     - List any files created or modified in this phase
     - STOP - do not proceed to next phase
   - **If no phase argument**:
     - Report full implementation summary across all phases

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/speckit.tasks` first to regenerate the task list.

## Usage Examples

```bash
/speckit.implement                 # Run all phases (may hit context limits)
/speckit.implement phase:1         # Setup only
/speckit.implement phase:2         # Foundational only
/speckit.implement phase:3         # User Story 1 only
/speckit.implement phase:us1       # Same as phase:3
/speckit.implement continue        # Resume from first incomplete task
/speckit.implement phase:polish    # Final polish phase only
```

**Recommended workflow for large features**:
1. Window 1: `/speckit.implement phase:1`
2. Window 2: `/speckit.implement phase:2`
3. Window 3: `/speckit.implement phase:3`
4. ... continue in fresh windows to avoid context overflow
