---
name: speckit-tasks
description: Use Spec Kit prompts from Codex CLI to generate an actionable task list from spec.md and plan.md. Use when you need implementation tasks with clear sequencing and ownership hints.
---

# Spec-Kit Tasks (Codex CLI)

Generate actionable tasks using `.codex/prompts/speckit.tasks.md`.

## When to Use

- spec.md and plan.md are ready
- You need a structured task list
- Preparing for implementation or issue creation

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.tasks.md`

## Run Prompt

Set the input that would follow `/speckit.tasks` and inject it into the prompt:

```bash
INPUT="Generate tasks for the approved plan"

PROMPT="$(cat .codex/prompts/speckit.tasks.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Next Steps

- `speckit-implement` to execute tasks
- `speckit-taskstoissues` to convert tasks into issues

## Related Skills

- `speckit-plan`
- `speckit-implement`
- `speckit-taskstoissues`
