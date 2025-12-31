# ai-coding-agent-skills

Cross-platform agent skills and prompts for AI coding assistants.

## Overview

This repository provides reusable skills and templates for multiple agent runtimes:

- **Claude Code** - Skills in `.claude/skills/`, agents in `.claude/agents/`, commands in `.claude/commands/`
- **Codex CLI** - Skills in `.codex/skills/` (claude-\*), prompts in `.codex/prompts/`
- **GitHub Copilot CLI** - Via symlinks in `.codex/skills/` and `.github/skills/`
- **GitHub Actions** - Agent files in `.github/agents/`, prompt files in `.github/prompts/`, skills in `.github/skills/`
- **Spec Kit** - Spec-Driven Development workflow skills (`speckit-*`) across all runtimes

Each skill directory contains a `skill.yaml` configuration and `SKILL.md` documentation.

### Spec Kit Workflow

This repository implements the **Spec-Driven Development** methodology via Spec Kit skills. The canonical workflow:

1. **Constitution** → Define project principles
2. **Specify** → Capture feature requirements (what/why)
3. **Clarify** (optional) → Resolve ambiguities
4. **Plan** → Create technical strategy (how)
5. **Analyze** (optional) → Validate consistency
6. **Tasks** → Generate ordered work items
7. **Implement** → Execute development

See **[AGENTS.md](./AGENTS.md#spec-kit-workflow)** for the complete workflow guide with examples and best practices.

## Quick start

1. Clone the repo:

   ```bash
   git clone git@github.com:dceoy/ai-coding-agent-skills.git
   ```

2. Pick a runtime and explore the skills:
   - **Claude Code:** `.claude/skills/` (skill directories), `.claude/agents/` (agent definitions), `.claude/commands/` (command prompts)
   - **Codex CLI:** `.codex/skills/` (skill directories), `.codex/prompts/` (prompt files)
   - **GitHub Actions:** `.github/agents/`, `.github/prompts/`, `.github/skills/`

3. Open a skill directory and read the `SKILL.md` to learn how to invoke it.

## Skills by runtime

### Claude Code

**Skills** (`.claude/skills/`)

- `copilot-ask`, `copilot-exec`, `copilot-review` - GitHub Copilot CLI integration
- `codex-ask`, `codex-exec`, `codex-review` - OpenAI Codex CLI integration
- `speckit-analyze`, `speckit-checklist`, `speckit-clarify`, `speckit-constitution`, `speckit-implement`, `speckit-plan`, `speckit-specify`, `speckit-tasks`, `speckit-taskstoissues` - Spec Kit workflow

**Agents** (`.claude/agents/`)

- `codex-ask.md`, `codex-exec.md`, `codex-review.md`, `codex-search.md` - Codex CLI agent definitions
- See [AGENTS.md](./AGENTS.md#codex-cli-agents) for agent documentation

**Commands** (`.claude/commands/`)

- `speckit.*.md` - Command prompts for Spec Kit workflow

### Codex CLI

**Skills** (`.codex/skills/`)

- `claude-ask`, `claude-exec`, `claude-review` - Claude Code integration (native)
- `copilot-*`, `speckit-*` - Symlinks to `.claude/skills/`

**Prompts** (`.codex/prompts/`)

- `speckit.*.md` - Prompt files for Spec Kit workflow

### GitHub Actions

**Agents** (`.github/agents/`)

- `speckit.*.agent.md` - Agent definitions for GitHub automation

**Prompts** (`.github/prompts/`)

- `speckit.*.prompt.md` - Prompt files for GitHub automation

**Skills** (`.github/skills/`)

- Symlinks to both `.claude/skills/` and `.codex/skills/`

## Structure

```
.
├── .claude/
│   ├── agents/          # Claude Code agent definitions (codex-*)
│   ├── commands/        # Claude Code command prompts (speckit.*)
│   └── skills/          # Claude Code skill directories (copilot-*, codex-*, speckit-*)
├── .codex/
│   ├── prompts/         # Codex CLI prompt files (speckit.*)
│   └── skills/          # Codex CLI skills (claude-* native, others symlinked)
├── .github/
│   ├── agents/          # GitHub automation agents (speckit.*.agent.md)
│   ├── prompts/         # GitHub workflow prompts (speckit.*.prompt.md)
│   ├── skills/          # GitHub skills (symlinks to .claude and .codex)
│   └── workflows/       # CI workflows (ci.yml)
├── .serena/             # Serena MCP memories, cache, and project config
│   ├── cache/
│   ├── memories/
│   └── project.yml
└── .specify/            # Spec Kit templates and memory files
    ├── memory/
    ├── scripts/
    └── templates/       # spec, plan, tasks, checklist, agent-file templates
```

## Prerequisites

Install and authenticate the required CLI tools before running skills:

- **Claude Code** - For `.claude/` skills, agents, and commands
  - Install: https://claude.com/claude-code
  - Auth: Follow CLI onboarding flow
- **GitHub Copilot CLI** - For `copilot-*` skills
  - Install: https://github.com/features/copilot/cli
  - Auth: `gh auth login` (requires GitHub Copilot subscription)
- **OpenAI Codex CLI** - For `codex-*` skills
  - Install: https://developers.openai.com/codex/cli/
  - Auth: ChatGPT subscription or API key in `~/.codex/config.toml`
- **Spec Kit** - Implemented via skills in this repository (no separate installation)

## Usage notes

- Skills do not always auto-run; use your agent's skill invocation flow or ask for the skill explicitly.
- If a skill fails, open its `SKILL.md` and verify prerequisites and command syntax.
- For Spec Kit workflows, see [AGENTS.md](./AGENTS.md#spec-kit-workflow) for the canonical command sequence and usage guide.

### Quick Spec Kit Example

```bash
# 1. First time: establish project principles
/speckit.constitution

# 2. For each feature: specify what to build
/speckit.specify Add user authentication with email login

# 3. Plan how to build it
/speckit.plan I'm using Node.js with Express and PostgreSQL

# 4. Break into tasks
/speckit.tasks

# 5. Implement
/speckit.implement
```

## Troubleshooting

**Skill not found**

- Confirm the skill directory exists in the expected runtime location
- Check that skill name matches exactly (case-sensitive)
- Verify `skill.yaml` exists in the skill directory

**CLI not in PATH**

- Ensure the tool is installed and accessible: `which <tool-name>`
- Add the tool's bin directory to your shell PATH
- Restart your terminal after installation

**Authentication errors**

- Re-run the tool's auth command:
  - Claude Code: Follow onboarding flow
  - Copilot CLI: `gh auth login`
  - Codex CLI: `codex` (follow auth flow) or configure `~/.codex/config.toml`
- Verify active subscription (Copilot, ChatGPT) or API key (Codex)

**Symlink issues**

- Some skills use symlinks (`.codex/skills/`, `.github/skills/`)
- If broken, verify source directories exist in `.claude/skills/`
- On Windows, ensure symlink support is enabled

## Contributing

See `AGENTS.md` for repository guidelines and agent-specific rules.

## License

See `LICENSE` for details.
