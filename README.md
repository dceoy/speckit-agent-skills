# cli-agent-skills

Cross-platform agent skills and prompts for AI coding assistants.

## Overview

This repository provides reusable skills and templates for multiple agent runtimes:

- Claude Code
- GitHub Copilot CLI
- Codex CLI
- Spec Kit

Each skill lives in a small folder with a `SKILL.md` that documents how to run it.

## Quick start

1. Clone the repo:

   ```bash
   git clone git@github.com:dceoy/cli-agent-skills.git
   ```

2. Pick a runtime and open a skill's `SKILL.md`:
   - Claude Code skills: `.claude/skills/`
   - Codex CLI skills: `.codex/skills/`

3. Follow the instructions in that `SKILL.md` to invoke the skill from your agent.

## Skills by runtime

### Claude Code (in `.claude/skills/`)

- `copilot-ask`, `copilot-exec`, `copilot-review`
- `codex-ask`, `codex-exec`, `codex-review`
- `speckit-*` workflow skills

### Codex CLI (in `.codex/skills/`)

- `claude-ask`, `claude-exec`, `claude-review`
- `copilot-ask`, `copilot-exec`, `copilot-review`
- `speckit-*` workflow skills

## Structure

```
.
├── .claude/
│   ├── agents/          # Claude Code agent definitions
│   ├── commands/        # Claude Code command prompts (Spec Kit)
│   └── skills/          # Claude Code skill definitions
├── .codex/
│   ├── prompts/         # Codex CLI prompt content (mirrors Spec Kit)
│   └── skills/          # Codex CLI skill definitions
├── .github/
│   ├── agents/          # GitHub automation agents
│   ├── prompts/         # GitHub workflow prompts
│   └── workflows/       # CI workflows
└── .specify/            # Spec Kit templates and memory files
```

## Prerequisites

You need the matching CLI tool installed and authenticated before running a skill.
Refer to each tool's official documentation for installation and login steps.

- Claude Code CLI
- GitHub Copilot CLI
- Codex CLI
- Spec Kit

## Usage notes

- Skills do not always auto-run; use your agent's skill invocation flow or ask for the skill explicitly.
- If a skill fails, open its `SKILL.md` and verify prerequisites and command syntax.
- For Spec Kit, follow the skill sequence described in the `speckit-*` skills.

## Troubleshooting

- Skill not found: confirm the skill directory exists and the name matches the request.
- CLI not in PATH: ensure the tool installs into your shell PATH.
- Auth errors: re-run the tool's login/auth command per its docs.

## Contributing

See `AGENTS.md` for repository guidelines and agent-specific rules.

## License

See `LICENSE` for details.
