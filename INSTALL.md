# Installation

## Codex Skill (Recommended for Codex users)

This repo includes a Codex skill at `.codex/skills/speckit`. Codex loads repo skills automatically.

Example usage:

```text
Use speckit sdd-flow for: Add user authentication with OAuth2
Use speckit.implement for specs/001-user-auth/
```

The Codex workflow is sequential (no subagents) and writes `AGENTS.md` for feature context.

## Install via Marketplace

```bash
# Add the marketplace
/plugin marketplace add kargarisaac/speckit-plugin

# Install the plugin
/plugin install speckit
```

## Update

```bash
/plugin update speckit
```

## Verify

After installation, you'll have access to:
- `/sdd-flow` - Complete spec-driven workflow
- `/build` - Multi-agent implementation
- `/speckit.implement` - Phase-by-phase implementation
- `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, etc.
