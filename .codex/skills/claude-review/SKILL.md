---
name: claude-review
description: Perform code reviews using Claude Code CLI to identify bugs, security vulnerabilities, performance issues, and code quality problems. Use when the user asks to review code, check for issues, security audit, or before committing. Requires Claude Code CLI installed.
---

# Claude Review Skill

Use Claude Code CLI to perform automated code reviews that identify issues and suggest improvements. This is a **read-only** analysis skill.

## When to Use

- User asks to "review" code
- User wants to check for bugs or issues
- User mentions "security", "performance", or "quality"
- Before committing code
- During pull request review
- User asks "what's wrong with this code?"

## Prerequisites

Verify Claude Code CLI is available:

```bash
claude --version  # Should display installed version
```

## Basic Usage

### Step 1: Determine Scope

Decide what to review:

- Uncommitted changes
- Specific file(s)
- Last commit
- Entire codebase

### Step 2: Check Current State

```bash
git status
git diff --stat
git diff
```

### Step 3: Execute Claude Code Review

Run Claude Code with a review-focused prompt (use `-p`/`--print` to output once and exit):

```bash
claude -p "Perform comprehensive code review of [SCOPE].

Check for:
1. CRITICAL ISSUES (must fix): security, runtime errors, data loss, breaking changes
2. IMPORTANT ISSUES (should fix): logic bugs, performance, error handling, type safety
3. SUGGESTIONS (consider): refactors, readability, docs

For each issue:
- Severity (Critical/Important/Suggestion)
- File path and line number
- Why it's a problem
- How to fix it

Do NOT make any changes - this is review only."
```

If your CLI does not support `-p`, run `claude` and paste the prompt.

### Step 4: Present Findings

Organize results by severity and include file references.

## Example Reviews

```bash
claude -p "Review all uncommitted changes for bugs, security issues, and performance problems. Do NOT modify code."
```

```bash
claude -p "Security review of src/auth/*.ts focusing on SQL injection, XSS, auth/authorization, secrets, and input validation. Do NOT modify code."
```

## Best Practices

- Categorize by severity
- Include specific file paths and line numbers
- Explain why each issue matters
- Verify findings to avoid false positives

## Error Handling

- If Claude Code CLI is not available, ensure it's installed and in your PATH
- If the review is too broad, narrow the scope
- If issues are unclear, request clarification or smaller scope

## Related Skills

- `claude-ask` for understanding code before reviewing
- `claude-exec` for fixing issues found in review

## Limitations

- Static analysis only
- May generate false positives
- Cannot run tests or execute code
