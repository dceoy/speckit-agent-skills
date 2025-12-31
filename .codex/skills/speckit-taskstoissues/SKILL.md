---
name: speckit-taskstoissues
description: Use Spec Kit prompts from Codex CLI to convert a task list into tracker issues (GitHub, etc.). Use when you want to turn speckit tasks into issues with titles and descriptions.
---

# Spec-Kit Tasks to Issues (Codex CLI)

Convert tasks into issues using `.codex/prompts/speckit.taskstoissues.md`.

## When to Use

- Task list is approved
- You want issues created from tasks
- Preparing for assignment and tracking

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.taskstoissues.md`

## Run Prompt

Set the input that would follow `/speckit.taskstoissues` and inject it into the prompt:

```bash
INPUT="Convert tasks into GitHub issues"

PROMPT="$(cat .codex/prompts/speckit.taskstoissues.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Related Skills

- `speckit-tasks`
