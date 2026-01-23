# Speckit — Spec-Driven Development Skill

Build high-quality software faster with a specification-first workflow. Speckit is based on [GitHub's spec-kit](https://github.com/github/spec-kit) with modifications and production-tested improvements from [Ashpreet Bedi's methodology](https://x.com/ashpreetbedi/status/2011220028453241218).

## What You Get

- Spec-first flow: clarify → plan → tasks → implement
- ADRs (`decisions.md`) to capture the "why"
- Progress tracking in `tasks.md` for session continuity
- Quality analysis and checklists for spec readiness

## Install the Skill

Copy or symlink `speckit/` into your tool's skill directory:

```bash
# Codex
ln -s "$(pwd)/speckit" ~/.codex/skills/speckit

# Claude Code
ln -s "$(pwd)/speckit" ~/.claude/skills/speckit
```

For other tools (opencode, etc.), place `speckit/` in their supported skills directory.

## Usage

```text
Use speckit sdd-flow for: Add user authentication with OAuth2
```

Implementation is sequential by default:

```text
Use speckit build for specs/001-user-auth/
```

Feature context is updated in existing agent context files (if present).

## Command Index

- `sdd-flow "description"` — complete workflow
- `speckit.specify` — create spec
- `speckit.clarify` — resolve ambiguities
- `speckit.plan` — generate implementation plan
- `speckit.tasks` — generate tasks
- `build` — execute implementation phase-by-phase (parallel workers supported in Codex and Claude Code)
- `speckit.analyze` — cross-artifact analysis
- `speckit.checklist` — requirements quality checklist
- `speckit.taskstoissues` — GitHub issues from tasks

## Output Structure

```
your-project/
├── specs/
│   └── ###-feature-name/
│       ├── spec.md
│       ├── plan.md
│       ├── tasks.md
│       ├── decisions.md
│       ├── research.md
│       ├── data-model.md
│       ├── quickstart.md
└── (optional) existing agent context files updated if present
```

## License

MIT
