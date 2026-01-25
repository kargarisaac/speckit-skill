---
name: speckit
description: Spec-driven development workflow (Speckit) for generating and maintaining specs, decisions, plans, tasks, and implementation guidance in repos that use specs/###-feature directories.
---

# Speckit spec-driven development

## Single command file

- The ONLY command file is `speckit.md`.
- All command routing and rules live there.

## Subcommands

- `new <feature description>` (default when no subcommand provided)
- `spec <feature folder>`
- `plan <feature folder>`
- `tasks <feature folder>`
- `implement <feature folder>`

## Question policy (strict)

- Ask questions only at the start and only for `spec.md`.
- Ask up to 5 questions total, all at once.
- After `spec.md` is written, do not ask more questions.

## Workflow (high level)

- Resolve `SPECKIT_ROOT` and ensure repo `specs/` exists.
- Run a targeted repo exploration for the feature context.
- Ask clarification questions (only if writing `spec.md`).
- Generate/update artifacts per the selected subcommand.

## Repo exploration

- Do a comprehensive, feature-focused scan of relevant code and configs first.
- Treat docs as secondary and verify against code when possible.
- After user answers, perform additional exploration if needed to close gaps.

## Set the skill root

- Set `SPECKIT_ROOT` to the absolute path of the Speckit skill folder.
- Prefer a repo-local `speckit/` at repo root if present.
- Otherwise use the user-installed skill at `~/.codex/skills/speckit` (the folder containing this `SKILL.md`).
- Use `scripts/` and `assets/templates/` from `SPECKIT_ROOT/`.
- Ensure the repo has a top-level `specs/` folder; create it if missing.

## Scripts

- Use `scripts/create-new-feature.sh` to create the feature folder and the four artifacts.

## Templates

- Only these templates are used:
  - `assets/templates/spec-template.md`
  - `assets/templates/decisions-template.md`
  - `assets/templates/plan-template.md`
  - `assets/templates/tasks-template.md`
