---
name: speckit-analyze
description: Use Spec Kit prompts from Codex CLI to analyze a spec/plan for risks, gaps, and feasibility. Use when you need a critical review before or after implementation.
---

# Spec-Kit Analyze (Codex CLI)

Analyze the spec and plan using `.codex/prompts/speckit.analyze.md`.

## When to Use

- You need a risk/feasibility review
- You want to identify gaps before implementation
- You want a post-implementation check

## Prerequisites

- Codex CLI installed: `codex --version`
- Spec Kit initialized: `.specify/` exists
- Prompt file present: `.codex/prompts/speckit.analyze.md`

## Run Prompt

Set the input that would follow `/speckit.analyze` and inject it into the prompt:

```bash
INPUT="Analyze the spec and plan for risks and missing details"

PROMPT="$(cat .codex/prompts/speckit.analyze.md)"
PROMPT="${PROMPT//\$ARGUMENTS/$INPUT}"

codex exec "$PROMPT"
```

If the input contains quotes or `$`, paste the text into the prompt manually instead of doing replacement.

## Related Skills

- `speckit-plan`
- `speckit-clarify`
