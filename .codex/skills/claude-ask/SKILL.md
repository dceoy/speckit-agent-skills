---
name: claude-ask
description: Ask Claude Code CLI questions about code to understand implementations, architecture, patterns, and debugging. Use when the user asks how code works, where something is implemented, what patterns are used, or needs read-only understanding. Requires Claude Code CLI installed.
---

# Claude Ask Skill

Use Claude Code CLI to answer questions about code without making modifications. This is a **read-only** analysis skill.

## When to Use

- User asks "how does X work?"
- User wants to find where something is implemented
- User needs to understand architecture or patterns
- User is debugging and needs to understand code flow
- User asks "what does this code do?"

## Prerequisites

Verify Claude Code CLI is available:

```bash
claude --version  # Should display installed version
```

## Basic Usage

### Step 1: Parse the Question

Extract what the user wants to know and the scope (files, feature, component).

### Step 2: Execute Claude Code Query

Run Claude Code in read-only mode (use `-p`/`--print` to output once and exit):

```bash
claude -p "Answer this question about the codebase: [QUESTION]

Provide:
1. Direct answer to the question
2. Specific file paths and line numbers
3. Code examples from the actual codebase
4. Related concepts or dependencies

Do NOT make any changes - this is read-only analysis."
```

If your CLI does not support `-p`, run `claude` and paste the prompt.

### Step 3: Present Answer

Format the response with:

- Summary (1-2 sentences)
- Details (explanation)
- File References (paths + line numbers)
- Code Examples
- Related Info (dependencies, gotchas)

## Example Queries

```bash
claude -p "Where is email validation implemented? Show all locations with file paths and line numbers. Do NOT modify code."
```

```bash
claude -p "Explain how authentication works. Include all files involved, the complete flow, and security measures. Do NOT modify code."
```

## Best Practices

- Always include file paths with line numbers
- Show actual code from the codebase
- Explain "why" not just "what"
- Verify paths and line numbers after the response

## Error Handling

- If Claude Code CLI is not available, ensure it's installed and in your PATH
- If authentication fails, ask the user to sign in via the CLI
- If the answer is unclear, ask a more specific question

## Related Skills

- `claude-exec` for code modifications
- `claude-review` for code reviews

## Limitations

- Read-only analysis
- Cannot run tests or execute code
- Limited by current codebase context
