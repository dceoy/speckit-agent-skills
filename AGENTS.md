# Repository Guidelines

## Project Structure & Module Organization

- `skills/` is the canonical source for shared skills.
- Each skill lives in `skills/<kebab-name>/SKILL.md` and includes YAML front matter.
- Runtime-specific entry points live in `.claude/commands/`, `.codex/prompts/`, `.github/agents/`, `.github/prompts/`, and `.gemini/commands/`.
- Keep filenames aligned across runtimes, for example `speckit.plan.md`, `speckit.plan.prompt.md`, and `speckit.plan.toml`.
- Spec Kit support files live in `.specify/scripts/bash/` and `.specify/templates/`.
- GitHub automation is defined in `.github/workflows/`.
- `CLAUDE.md` is a symlink to this file, so edit `AGENTS.md` directly.

## Build, Test, and Development Commands

- There is no compile step in this repository; most changes are Markdown, templates, and Bash helpers.
- `git status -sb` shows the current workspace and changed runtime files.
- `git diff --check` catches trailing whitespace and malformed patches before commit.
- `bash .specify/scripts/bash/check-prerequisites.sh --help` lists supported validation modes.
- `bash .specify/scripts/bash/check-prerequisites.sh --paths-only` prints the active feature paths without modifying files.
- `bash .specify/scripts/bash/update-agent-context.sh codex` refreshes agent context from the current `specs/<id>-<name>/plan.md`.

## Coding Style & Naming Conventions

- Use concise Markdown with ATX headings and direct, imperative instructions.
- Keep skill directories in kebab-case and limit each directory to one `SKILL.md`.
- Preserve stable front matter keys such as `name` and `description`.
- For Bash, follow the existing style in `.specify/scripts/bash/*.sh`: `#!/usr/bin/env bash`, quoted variables, small functions, and defensive checks.
- When you add or rename a skill, update the matching runtime wrapper filenames in every supported directory.

## Testing Guidelines

- No dedicated unit-test framework or coverage gate is checked into this repo.
- Validate changes by smoke-testing Bash helpers with `--help`, `--json`, or `--paths-only`.
- Verify Markdown renders cleanly.
- Confirm runtime-specific prompt or agent files stay synchronized with the source skill.

## Commit & Pull Request Guidelines

- Format Markdown files with `npx prettier --write` and lint Bash scripts with `shellcheck` before committing or creating a PR.
- If you change `.github/workflows/`, lint workflow files with `actionslint`, `yamllint`, `zizmor --fix=safe`, and `checkov --framework=github_actions,github_configuration`.
- Keep PRs focused and include: concise summary, affected workflow paths, and linked issue/context.
- Branch names use appropriate prefixes on creation (e.g., `feature/...`, `bugfix/...`, `refactor/...`, `docs/...`, `chore/...`).
- When instructed to create a PR, create it as a draft with appropriate labels by default.

## Spec Kit Workflow

- Keep workflow content synchronized across skills, prompts, and templates.
- The canonical order in this repo is `constitution -> specify` or `baseline -> clarify -> plan -> tasks -> analyze -> implement`.
- If you add or rename a step, update the source skill and every affected runtime wrapper in the same change.
