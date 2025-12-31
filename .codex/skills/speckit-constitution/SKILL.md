---
name: speckit-constitution
description: Use Spec Kit prompts from Codex CLI to update or generate the project constitution for Spec Kit workflows. Use when you need to set or revise governing principles and constraints.
---

# Spec-Kit Constitution (Codex CLI)

Generate or update the constitution using `.codex/prompts/speckit.constitution.md`.

## When to Use

- Establishing or updating project constitution
- You need governing principles, constraints, or guardrails

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.constitution.md`

## Run Prompt

Set the input that would follow `/speckit.constitution` and inject it into the prompt:

```bash
INPUT="Update the constitution for this repository"

PROMPT="$(cat .codex/prompts/speckit.constitution.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Related Skills

- `speckit-specify`
- `speckit-plan`
