# CLAUDE.md — [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]

## Project Context

[2-3 sentence description of what this feature does and why it matters]

## Before Starting Work

1. Read `specs/[###-feature-name]/spec.md` for requirements
2. Check `specs/[###-feature-name]/tasks.md` for current progress (see Progress section)
3. Review `specs/[###-feature-name]/decisions.md` for architectural decisions made
4. Look at existing similar implementations for patterns

## Related Files

### Files to Modify
- `path/to/file1` - [What changes needed]
- `path/to/file2` - [What changes needed]

### Files to Create
- `path/to/new/file1` - [Purpose]
- `path/to/new/file2` - [Purpose]

### Reference Implementations
- `path/to/similar/` - Follow this pattern for [component/service]

## Code Patterns

[Describe specific patterns to follow for this feature]

- Pattern 1: [Description]
- Pattern 2: [Description]

## Don't

- Don't add features not in spec.md
- Don't break existing [specific protocol/interface]
- Don't skip tests for [critical components]
- [Feature-specific constraints]

## Session Continuity

When resuming work on this feature:
1. Check `tasks.md` Progress section for what's done/in-progress
2. Check `decisions.md` for any pending decisions
3. Run tests to verify current state: `[test command]`

## Quick Commands

```bash
# Run tests for this feature
[test command]

# Start multi-agent implementation (recommended)
/build specs/[###-feature-name]/

# Or phase-by-phase implementation
/speckit.implement phase:1
/speckit.implement continue
```
