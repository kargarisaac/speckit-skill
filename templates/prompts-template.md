# Reusable Prompts: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`
**Created**: [DATE]

## Purpose

Store reusable prompts for common tasks during this feature's development. Copy-paste these prompts to avoid retyping. Claude can also add new prompts as patterns emerge.

---

## Implementation Prompts

### Resume Implementation
```
Read specs/[###-feature-name]/tasks.md and check the Progress section.
Continue implementation from where we left off.
Mark each task [X] immediately after completing it.
Update the Progress section after each task.
```

### Sync Design Docs with Code
```
Compare the current implementation in [paths] with specs/[###-feature-name]/spec.md.
Identify any gaps between spec and implementation.
Update the spec if implementation intentionally diverged (document why in decisions.md).
```

### Review and Test
```
Read specs/[###-feature-name]/REVIEW_PROMPT.md and run a full review.
Check each requirement against the implementation.
Run all tests and report any failures.
```

---

## Testing Prompts

### Run Tests for This Feature
```
Run tests related to [feature area]:
- [specific test commands]

Report any failures with file paths and error messages.
```

### Add Missing Tests
```
Review the implementation in [paths].
Identify any untested code paths.
Add tests following the patterns in [test file reference].
```

---

## Debugging Prompts

### Investigate Issue
```
The [component] is [problem description].
Check:
1. [First thing to check]
2. [Second thing to check]
3. [Third thing to check]

Report findings and suggest fixes.
```

---

## Refactoring Prompts

### Cleanup After Implementation
```
Review the code in [paths] for:
- Dead code or unused imports
- Duplicated logic that could be extracted
- Missing error handling
- Type safety issues

Make minimal, focused improvements. One PR per concern.
```

---

## Custom Prompts

<!-- Add your own reusable prompts below -->

### [Prompt Name]
```
[Prompt content]
```

---

## Notes

- Add new prompts as patterns emerge during development
- Keep prompts focused and reusable
- Reference specific file paths when possible
- Claude writes these prompts too - just ask it to add useful ones
