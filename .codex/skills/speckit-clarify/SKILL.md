---
name: speckit-clarify
description: Use Spec Kit prompts from Codex CLI to clarify ambiguous requirements in an existing spec.md. Use when a spec has open questions, needs scope alignment, or requires clarification notes. Follow-up step after speckit-specify.
---

# Spec-Kit Clarify (Codex CLI)

Clarify open questions in an existing spec using `.codex/prompts/speckit.clarify.md`.

## When to Use

- spec.md includes open questions or assumptions
- Requirements are ambiguous or conflicting
- You need stakeholder-aligned clarifications

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.clarify.md`

## Run Prompt

Set the input that would follow `/speckit.clarify` and inject it into the prompt:

```bash
INPUT="Clarify remaining requirements and resolve open questions"

PROMPT="$(cat .codex/prompts/speckit.clarify.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Next Steps

- `speckit-plan` once clarifications are resolved

## Related Skills

- `speckit-specify`
- `speckit-plan`
