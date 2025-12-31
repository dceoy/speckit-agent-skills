---
name: speckit-checklist
description: Use Spec Kit prompts from Codex CLI to generate or update requirement and quality checklists for a feature. Use when you need verification criteria before planning or implementation.
---

# Spec-Kit Checklist (Codex CLI)

Generate checklists using `.codex/prompts/speckit.checklist.md`.

## When to Use

- You need requirement or quality checklists
- You want validation gates before planning or implementation

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.checklist.md`

## Run Prompt

Set the input that would follow `/speckit.checklist` and inject it into the prompt:

```bash
INPUT="Generate requirement and QA checklists for this feature"

PROMPT="$(cat .codex/prompts/speckit.checklist.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Related Skills

- `speckit-specify`
- `speckit-plan`
- `speckit-tasks`
