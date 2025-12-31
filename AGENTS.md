# Agent Repository Guidelines

Guidelines for editing agent definitions, skills, and prompts in this repository.

## Project Structure

- `.claude/agents/` Claude Code agent definitions (see `.claude/agents/README.md`).
- `.claude/commands/` Claude Code command prompts (Spec Kit commands).
- `.claude/skills/` Claude Code skill definitions (`copilot-*`, `codex-*`, `speckit-*`).
- `.codex/prompts/` Spec Kit prompt content for Codex CLI usage.
- `.codex/skills/` Codex CLI skill definitions (`claude-*`, `copilot-*`, `speckit-*`).
- `.github/agents/` and `.github/prompts/` GitHub automation agents/prompts.
- `.github/workflows/ci.yml` CI workflows (Claude review/bot runs on PRs).
- `.specify/` Spec Kit templates and memory files.
- `.serena/` Serena MCP memories and project configuration.
- Root docs: `README.md` (overview), `LICENSE`, `AGENTS.md`, and `CLAUDE.md`.

## Canonical Files

- `CLAUDE.md` is a symlink to `AGENTS.md`. Edit `AGENTS.md` only.

## Prerequisites

Install and authenticate the required CLI tools before running skills. See `README.md` for the current prerequisites and links.

## Coding Style & Naming

- Use Markdown for docs and prompts; keep headings short.
- Prefer fenced code blocks for examples.
- Keep filenames in kebab-case (e.g., `codex-ask.md`, `speckit.plan.md`).
- YAML uses 2-space indentation (see `.github/workflows/ci.yml`).
- Keep instructions concise and task-focused.

## Validation

- Review diffs with `git diff`.
- Check that Markdown renders cleanly.
- When adding/renaming agents or prompts, update any local indexes or READMEs that list them (for example, `.claude/agents/README.md`).

## Commit & PR Guidelines

- Commit messages use short, imperative summaries (e.g., "Add …", "Update …").
- PRs should describe the change and rationale.
- Avoid tagging automation users unless needed; CI reacts to PR events and `@claude` mentions.

## Agent-Specific Notes

- Agent and prompt files are the product of this repo—treat them as source of truth.
- Prefer small, focused edits per file for reviewability.

## Serena MCP Usage (Prioritize When Available)

- Use Serena MCP tools first when available.
- Check Serena MCP docs/help before calling a tool to confirm names and arguments.
- Use MCP-exposed tools for reading/writing files and fetching data.
- Never hardcode secrets; rely on environment variables or MCP credentials.
- If Serena MCP is unavailable or missing a capability, state it and propose a safe fallback.
- Be explicit and reproducible when describing tool usage.
