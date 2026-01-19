---
name: speckit
description: Spec-driven development workflow (Speckit) for generating and maintaining feature specs, plans, tasks, checklists, and implementation guidance in repos that use specs/###-feature directories. Use when requests mention sdd-flow or speckit commands (specify, clarify, plan, tasks, implement, analyze, checklist, taskstoissues) or ask for spec-first planning and build orchestration.
---

# Speckit spec-driven development

## Set the skill root

- Resolve the repo root (use `git rev-parse --show-toplevel` or the current working directory).
- Set `SPECKIT_ROOT` to the absolute path of the `speckit/` folder so scripts and templates resolve.
- Use `scripts/` and `assets/templates/` from this skill directory.

## Core artifacts

- Keep specs in `specs/###-feature-name/` with `spec.md`, `plan.md`, `tasks.md`, `decisions.md`, `research.md`, `data-model.md`, `quickstart.md`, `AGENTS.md`, `CLAUDE.md`, and optional `contracts/`.

## Workflow map

- For a single-command workflow request, follow `references/sdd-flow.md`.
- For individual phases, follow:
  - `references/speckit.specify.md`
  - `references/speckit.clarify.md`
  - `references/speckit.plan.md`
  - `references/speckit.tasks.md`
  - `references/speckit.analyze.md`
  - `references/speckit.checklist.md`
  - `references/speckit.implement.md`
  - `references/build.md`
  - `references/speckit.taskstoissues.md`

## Adaptations for Platform Capabilities

- If the platform has a structured question tool (Claude: AskUserQuestion, OpenCode: question), use it; Codex should ask in chat with 1–2 questions at a time, preserving the constraints in the reference.
- If the platform supports parallel agents/workers, use them; otherwise do sequential passes and keep outputs distinct to simulate multi-agent separation.
- When a reference says "use browser automation", use the available tool(s) or skip if not required.
- Keep all file writes local; do not use external services unless requested.

## Scripts

- Use `scripts/create-new-feature.sh` to create the specs directory and feature branch, following the numbering and short-name rules in the references.
- Use `scripts/check-prerequisites.sh` and `scripts/setup-plan.sh` to locate feature paths and ensure templates.
- Use `scripts/update-agent-context.sh` only when updating agent context files is requested; do not modify unrelated agent files.

## Templates

- Use `assets/templates/spec-template.md`, `plan-template.md`, `tasks-template.md`, `decisions-template.md`, `checklist-template.md`, `feature-agents-template.md`, and `agent-file-template.md` as scaffolding.
- Do not modify template content unless the user explicitly asks to change the workflow.

## Output expectations

- Maintain progress updates in `tasks.md` exactly as defined in the references.
- Keep requirements testable and technology-agnostic in `spec.md`.
- Keep decisions traceable via ADRs in `decisions.md`.
