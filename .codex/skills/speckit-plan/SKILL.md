---
name: speckit-plan
description: Use Spec Kit prompts from Codex CLI to generate a technical implementation plan (plan.md) from an existing spec.md. Use after speckit-specify and clarification to define architecture, data models, APIs, and technical decisions.
---

# Spec-Kit Plan (Codex CLI)

Generate the technical implementation plan using `.codex/prompts/speckit.plan.md`.

## When to Use

- spec.md is ready and clarified
- You need architecture, data model, and API plan
- Preparing for task breakdown and implementation

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.plan.md`

## Run Prompt

Set the input that would follow `/speckit.plan` and inject it into the prompt:

```bash
INPUT="I am building with React + Node + Postgres"

PROMPT="$(cat .codex/prompts/speckit.plan.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Next Steps

- `speckit-tasks` to generate the task list
- `speckit-analyze` for risk/complexity review

## Related Skills

- `speckit-specify`
- `speckit-clarify`
- `speckit-tasks`
- `speckit-analyze`
