---
name: claude-exec
description: Execute development tasks using Claude Code CLI for code generation, refactoring, feature implementation, and bug fixes. Use when the user asks to create code, add features, refactor, fix bugs, or generate tests. Requires Claude Code CLI installed.
---

# Claude Exec Skill

Use Claude Code CLI to execute development tasks that involve code modifications. This skill **modifies code**.

## When to Use

- User asks to "add", "create", "implement", or "generate" code
- User wants to refactor existing code
- User needs to fix a bug
- User wants to create tests
- User asks to "update" or "modify" code

## Prerequisites

Verify Claude Code CLI is available:

```bash
claude --version  # Should display installed version
```

## Basic Usage

### Step 1: Understand the Task

Parse what needs to be done:

- What to create/modify?
- Which files are affected?
- Expected outcome?
- Constraints or requirements?

### Step 2: Gather Context

Before executing, understand the current state:

```bash
git status
git diff
```

Read relevant files to follow existing patterns.

### Step 3: Execute Claude Code

Run Claude Code with clear instructions (use `-p`/`--print` to output once and exit):

```bash
claude -p "[TASK]

Follow these guidelines:
- Follow existing code patterns and conventions
- Add appropriate error handling
- Include necessary imports
- Maintain readability and code quality
- Add comments only for complex logic

Do NOT change unrelated files."
```

If your CLI does not support `-p`, run `claude` and paste the prompt.

### Step 4: Verify Changes

After execution:

```bash
git status
git diff
```

Run relevant checks or tests if available.

### Step 5: Report Results

Provide:

- Files modified/created
- Summary of changes
- Verification results
- Any issues or warnings

## Example Tasks

```bash
claude -p "Add email validation to ContactForm.tsx:
- Use regex: /^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/
- Show error message below input field
- Validate on blur and submit
- Prevent submission if invalid
- Style error message in red (#dc2626)
Follow existing form validation patterns."
```

```bash
claude -p "Refactor validation logic in src/components/LoginForm.tsx:
- Extract validation into src/utils/validation.ts
- Create validateEmail and validatePassword functions
- Keep existing behavior
- Add TypeScript types"
```

## Best Practices

- Be specific in task descriptions (paths, requirements, constraints)
- Review all changes with `git diff`
- Run lint/tests when available
- Avoid auto-approving risky changes

## Error Handling

- If execution fails, tighten scope and re-run
- If changes are incorrect, revert and re-run with clearer guidance
- If tests fail, fix failures before reporting success

## Related Skills

- `claude-ask` for understanding code before modifying
- `claude-review` for reviewing changes after modification

## Limitations

- Cannot run tests by itself
- May miss project-specific constraints
- Requires manual verification
