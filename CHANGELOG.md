# Changelog

All notable changes to the Speckit plugin will be documented in this file.

## [2.1.0] - 2026-01-15

### Added
- **New `/build` command**: Multi-agent orchestration for implementation
  - Lead agent orchestrates and tracks progress in tasks.md
  - Implementer subagents handle task batches with fresh context windows
  - Tester subagents verify functionality after each batch
  - Browser MCP (chrome-in-claude/chrome-devtools) for UI testing
  - Dynamic batch sizing based on task complexity
  - Retry once on failure, then ask user
  - Usage: `/build specs/001-feature/` or `/build --continue`

### Removed
- **Ralph Loop references**: All references to Ralph Loop have been removed
- **Prompts folder generation**: `prompts/` folder with implement.md, test.md, review.md no longer generated
- **IMPLEMENTATION_PROMPT.md**: No longer generated
- **REVIEW_PROMPT.md**: No longer generated
- **templates/prompts-template.md**: Deleted

### Notes
- `/speckit.implement` remains unchanged (phase-by-phase execution)
- `/build` is the new recommended way to implement features with multi-agent orchestration

## [2.0.0] - 2026-01-14

### Added
- **Architectural Decision Records (ADRs)**: `decisions.md` captures the "why" behind decisions
- **Reusable Prompts**: `prompts.md` for common development tasks
- **Per-Feature Instructions**: Feature-specific `CLAUDE.md` files
- **Progress Tracking**: Session continuity section in `tasks.md`
- **Runnable Examples**: Cookbooks section for verification

### Enhanced
- All templates now support Ashpreet Bedi's methodology
- Improved session continuity for context resets
- Better task organization with progress tracking

### Based On
- GitHub's spec-kit framework
- Ashpreet Bedi's production-tested improvements

## [1.0.0] - Initial Development

### Added
- Core spec-driven development workflow
- `/sdd-flow` orchestrator command
- `/speckit.specify` - Create specifications
- `/speckit.clarify` - Resolve ambiguities
- `/speckit.plan` - Generate implementation plans
- `/speckit.tasks` - Break down into tasks
- `/speckit.implement` - Execute implementation
- `/speckit.analyze` - Cross-artifact analysis
- `/speckit.checklist` - Quality validation
- `/speckit.constitution` - Project principles
- `/speckit.taskstoissues` - GitHub integration
