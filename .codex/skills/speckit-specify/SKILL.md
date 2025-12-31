---
name: speckit-specify
description: Use Spec Kit prompts from Codex CLI to create or update a feature specification from a natural language description. Use when starting a new feature, documenting requirements, or turning ideas into spec.md. First step in the Spec Kit workflow.
---

# Spec-Kit Specify (Codex CLI)

Create the feature spec from a natural language description using the Codex CLI prompt in `.codex/prompts/speckit.specify.md`.

## When to Use

- User describes a new feature to build
- Starting a new project or feature
- Need to document requirements in spec.md

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.specify.md`

## Run Prompt

Set the input that would follow `/speckit.specify` and inject it into the prompt:

```bash
INPUT="Add user authentication with email/password"

PROMPT="$(cat .codex/prompts/speckit.specify.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Next Steps

- `speckit-clarify` for open questions
- `speckit-plan` for the implementation plan
- `speckit-tasks` to generate tasks

## Related Skills

- `speckit-clarify`
- `speckit-plan`
- `speckit-tasks`
