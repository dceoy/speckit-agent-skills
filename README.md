# speckit-agent-skills

Agent skills for [Spec Kit](https://github.com/github/spec-kit)

## Overview

This repository provides reusable skills and templates for multiple agent runtimes:

- **Shared skills** - Source skills live in `skills/` and are symlinked to `.claude/skills`, `.codex/skills`, and `.github/skills`
- **Claude Code** - Spec Kit commands in `.claude/commands/` (skills via the symlinked `.claude/skills`)
- **Codex CLI** - Prompt files in `.codex/prompts/` (skills via the symlinked `.codex/skills`)
- **GitHub Copilot CLI** - Agent files in `.github/agents/`, prompt files in `.github/prompts/`, skills via `.github/skills`
- **Gemini CLI** - Command files in `.gemini/commands/`
- **Spec Kit** - Spec-Driven Development workflow skills (`speckit-*`) across all runtimes

Each skill directory has a `SKILL.md` with YAML front matter that includes the skill configuration and documentation.

### Spec Kit Workflow

This repository implements the **Spec-Driven Development** methodology via Spec Kit skills. The canonical workflow:

1. **Constitution** → Define project principles
2. **Specify** → Capture feature requirements (what/why)
   - Or **Baseline** → Generate specs from existing code
3. **Clarify** (optional) → Resolve ambiguities
4. **Plan** → Create technical strategy (how)
5. **Analyze** (optional) → Validate consistency
6. **Tasks** → Generate ordered work items
7. **Implement** → Execute development

See **[AGENTS.md](./AGENTS.md#spec-kit-workflow)** for the complete workflow guide with examples and best practices.

#### Visual workflow

```mermaid
flowchart TD
  %% Core (recommended order)
  C0["/speckit.constitution"] --> C1["/speckit.specify"] --> C2["/speckit.plan"] --> C3["/speckit.tasks"] --> C4["/speckit.implement"]

  %% Alternative entry for existing code
  C0 --> FC["/speckit.baseline"]
  FC --> C2

  %% Optional (dashed = insert/assist)
  C1 -.-> O1["/speckit.clarify"]
  FC -.-> O1
  O1 -.-> C2

  C3 -.-> O2["/speckit.analyze"]
  O2 -.-> C4

  C1 -.-> O3["/speckit.checklist"]
  C2 -.-> O3
  C3 -.-> O3
```

## Quick start

1. Clone the repo:

   ```bash
   git clone git@github.com:dceoy/speckit-agent-skills.git
   ```

2. Pick a runtime and explore the skills:
   - **Shared skills:** `skills/` (source directories with `SKILL.md` + YAML front matter)
   - **Claude Code:** `.claude/commands/` (command prompts), `.claude/skills` (symlink to `../skills`)
   - **Codex CLI:** `.codex/prompts/` (prompt files), `.codex/skills` (symlink to `../skills`)
   - **Gemini CLI:** `.gemini/commands/` (prompt files)
   - **GitHub Copilot CLI:** `.github/agents/` (Spec Kit agents), `.github/prompts/`, `.github/skills` (symlink to `../skills`)

3. Open a skill directory and read the `SKILL.md` to learn how to invoke it.

## Skills by runtime

### Shared skills (`skills/`)

- `speckit-*` - Spec Kit workflow skills

### Runtime access

- **Claude Code:** `.claude/commands/` (Spec Kit prompts) and `.claude/skills` (symlink to `../skills`)
- **Codex CLI:** `.codex/prompts/` (Spec Kit prompts) and `.codex/skills` (symlink to `../skills`)
- **GitHub Copilot CLI:** `.github/agents/` (Spec Kit agents), `.github/prompts/`, `.github/skills` (symlink to `../skills`)
- **Gemini CLI:** `.gemini/commands/` (Spec Kit prompts)

## Structure

```
.
├── skills/              # Source skills (speckit-*)
├── .claude/
│   ├── commands/        # Claude Code command prompts (speckit.*)
│   └── skills -> ../skills
├── .codex/
│   ├── prompts/         # Codex CLI prompt files (speckit.*)
│   └── skills -> ../skills
├── .gemini/
│   └── commands/        # Gemini CLI prompt files (speckit.*.toml)
├── .github/
│   ├── agents/          # GitHub Copilot CLI agents (speckit.*.agent.md)
│   ├── prompts/         # GitHub Copilot CLI prompts (speckit.*.prompt.md)
│   ├── skills -> ../skills
│   └── workflows/       # CI workflows (ci.yml)
└── .specify/            # Spec Kit templates and memory files
    ├── memory/
    ├── scripts/
    └── templates/       # spec, plan, tasks, checklist, agent-file templates
```

## Prerequisites

Install and authenticate the required CLI tools before running skills:

- **Claude Code** - For `.claude/commands/` and shared skills via `.claude/skills`
  - Install: https://claude.com/claude-code
  - Auth: Follow CLI onboarding flow
- **GitHub Copilot CLI** - For `.github/agents/` and shared skills via `.github/skills`
  - Install: https://github.com/features/copilot/cli
  - Auth: `gh auth login` (requires GitHub Copilot subscription)
- **OpenAI Codex CLI** - For `.codex/prompts/` and shared skills via `.codex/skills`
  - Install: https://developers.openai.com/codex/cli/
  - Auth: ChatGPT subscription or API key in `~/.codex/config.toml`
- **Gemini CLI** - For `.gemini/commands/`
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

# 2b. Or generate specs from existing code
/speckit.baseline src/auth/

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
- Verify `SKILL.md` exists in the skill directory and includes YAML front matter

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

- Skills are shared via symlinks (`.claude/skills/`, `.codex/skills/`, `.github/skills/`)
- If broken, verify `skills/` exists and the symlinks point to it
- On Windows, ensure symlink support is enabled

## Contributing

See [AGENTS.md](./AGENTS.md) for repository guidelines and agent-specific rules.

## License

See [LICENSE](./LICENSE) for details.
