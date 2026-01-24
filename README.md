# Speckit — Spec-Driven Development Skill

Build high-quality software faster with a specification-first workflow. Speckit is based on [GitHub's spec-kit](https://github.com/github/spec-kit) with modifications and production-tested improvements from [Ashpreet Bedi's methodology](https://x.com/ashpreetbedi/status/2011220028453241218).

## What You Get

- Spec-first flow: spec → decisions → plan → tasks → implement
- ADRs (`decisions.md`) to capture the "why"
- Four core artifacts per feature (`spec.md`, `decisions.md`, `plan.md`, `tasks.md`)

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
Use speckit new Add user authentication with OAuth2
```

Update artifacts individually:

```text
Use speckit spec specs/001-user-auth/
Use speckit plan specs/001-user-auth/
Use speckit tasks specs/001-user-auth/
Use speckit implement specs/001-user-auth/
```

## Command Index

- `new <feature description>` — create feature folder, spec, decisions, plan, tasks
- `spec <feature folder>` — update spec only (questions allowed here only)
- `plan <feature folder>` — update decisions + plan (no questions)
- `tasks <feature folder>` — update tasks only (no questions)
- `implement <feature folder>` — implement tasks in order, mark `[x]`

## Output Structure

```
your-project/
├── specs/
│   └── ###-feature-name/
│       ├── spec.md
│       ├── decisions.md
│       ├── plan.md
│       └── tasks.md
```

## License

MIT
