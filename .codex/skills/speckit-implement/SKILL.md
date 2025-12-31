---
name: speckit-implement
description: Use Spec Kit prompts from Codex CLI to implement tasks generated from spec.md and plan.md. Use when you are ready to execute the task list and apply code changes.
---

# Spec-Kit Implement (Codex CLI)

Execute tasks using `.codex/prompts/speckit.implement.md`.

## When to Use

- Task list is ready
- You want to apply code changes for the feature
- Moving from planning to implementation

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.implement.md`

## Run Prompt

Set the input that would follow `/speckit.implement` and inject it into the prompt:

```bash
INPUT="Implement the approved tasks in order"

PROMPT="$(cat .codex/prompts/speckit.implement.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Next Steps

- Run tests and review changes
- `speckit-analyze` if you need a post-implementation review

## Related Skills

- `speckit-tasks`
- `speckit-analyze`
