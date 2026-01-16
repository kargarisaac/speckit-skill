---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
handoffs: 
  - label: Create Tasks
    agent: speckit.tasks
    prompt: Break the plan into tasks
    send: true
  - label: Create Checklist
    agent: speckit.checklist
    prompt: Create a checklist for the following domain...
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `${SPECKIT_ROOT}/scripts/setup-plan.sh --json` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `.specify/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 0: Generate decisions.md (capture ADRs from research decisions)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update feature AGENTS.md with concrete file paths
   - Phase 1: Update agent context by running the agent script
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts including decisions.md and updated AGENTS.md.

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

4. **Generate decisions.md** using `${SPECKIT_ROOT}/assets/templates/decisions-template.md`:
   - Convert each major decision from research.md into an ADR
   - Each ADR should include: Context, Options Considered, Decision, Consequences
   - Number ADRs sequentially (ADR-001, ADR-002, etc.)
   - Add any pending decisions that need resolution during implementation
   - This captures the "why" for future sessions and contributors

**Output**: research.md with all NEEDS CLARIFICATION resolved, decisions.md with ADRs

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Update feature AGENTS.md** with concrete file paths:
   - Read `SPECS_DIR/AGENTS.md` (created by speckit.specify)
   - Update "Files to Modify" with actual files to change
   - Update "Files to Create" with new files to add
   - Update "Reference Implementations" with similar code patterns found
   - Add feature-specific code patterns from research
   - Update "Don't" rules based on decisions made

4. **Agent context update**:
   - Run `${SPECKIT_ROOT}/scripts/update-agent-context.sh codex`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/*, quickstart.md, AGENTS.md (updated), agent-specific file

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
