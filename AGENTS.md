# Agent Repository Guidelines

Guidelines for editing agent definitions, skills, and prompts in this repository.

## Project Structure

### Claude Code Runtime (`.claude/`)

- `.claude/agents/` - Agent definitions for autonomous task execution (codex-ask, codex-exec, codex-review, codex-search)
- `.claude/commands/` - Command prompt files for Spec Kit workflow (speckit.\*.md)
- `.claude/skills/` - Skill directories with skill.yaml + SKILL.md (copilot-_, codex-_, speckit-\*)

### Codex CLI Runtime (`.codex/`)

- `.codex/prompts/` - Prompt files for Spec Kit workflow (speckit.\*.md)
- `.codex/skills/` - Native claude-\* skills + symlinks to .claude/skills/ for others

### GitHub Actions Runtime (`.github/`)

- `.github/agents/` - Agent definitions for GitHub automation (speckit.\*.agent.md)
- `.github/prompts/` - Prompt files for GitHub workflows (speckit.\*.prompt.md)
- `.github/skills/` - Symlinks to both .claude/skills/ and .codex/skills/
- `.github/workflows/` - CI workflow definitions (ci.yml)

### Spec Kit Framework (`.specify/`)

- `.specify/templates/` - Templates for spec, plan, tasks, checklist, agent files
- `.specify/memory/` - Constitution and other persistent memory
- `.specify/scripts/` - Helper scripts

### Serena MCP (`.serena/`)

- `.serena/memories/` - MCP memory files
- `.serena/cache/` - MCP cache
- `.serena/project.yml` - Project configuration

### Root Documentation

- `README.md` - Repository overview and quick start
- `AGENTS.md` - This file (agent guidelines + Spec Kit workflow)
- `CLAUDE.md` - Symlink to AGENTS.md (do not edit directly)
- `LICENSE` - Repository license

## Canonical Files

- `CLAUDE.md` is a symlink to `AGENTS.md`. Edit `AGENTS.md` only.

## Prerequisites

Install and authenticate the required CLI tools before running skills. See `README.md` for the current prerequisites and links.

## Coding Style & Naming

**Markdown Files**

- Use Markdown for docs, prompts, and agent definitions
- Keep headings short and descriptive
- Prefer fenced code blocks for examples
- Keep instructions concise and task-focused

**File Naming**

- Skills: `<tool>-<action>` directories (e.g., `codex-ask/`, `speckit-plan/`)
- Agents: `<tool>-<action>.md` (e.g., `codex-ask.md`)
- Commands: `<tool>.<action>.md` (e.g., `speckit.plan.md`)
- Prompts: `<tool>.<action>.prompt.md` or `<tool>.<action>.agent.md`
- Use kebab-case for multi-word names

**YAML Files**

- 2-space indentation (see `.github/workflows/ci.yml`)
- Skill configs use `skill.yaml` in each skill directory
- Boolean values: `true`/`false` (lowercase)

**Skill Structure**
Each skill directory must contain:

- `skill.yaml` - Skill configuration and metadata
- `SKILL.md` - Documentation and usage instructions

## Symlink Strategy

This repository uses symlinks to share skills across runtimes:

**Source Skills** (`.claude/skills/`)

- Primary location for copilot-_, codex-_, and speckit-\* skills
- Contains actual skill directories with skill.yaml and SKILL.md

**Symlinked Skills**

- `.codex/skills/` - Symlinks to copilot-_ and speckit-_ from .claude/skills/
- `.github/skills/` - Symlinks to all skills from both .claude and .codex

**Benefits**

- Single source of truth for shared skills
- Easy maintenance (edit once, works everywhere)
- Reduced duplication

**When Adding/Modifying Skills**

1. Create/edit skills in their primary location:
   - claude-\* → `.codex/skills/` (native)
   - copilot-_, codex-_, speckit-\* → `.claude/skills/` (native)
2. Create symlinks as needed:
   ```bash
   ln -s ../../.claude/skills/speckit-example .codex/skills/speckit-example
   ln -s ../../.claude/skills/speckit-example .github/skills/speckit-example
   ```
3. Verify symlinks work: `ls -la .codex/skills/`

## Validation

- Review diffs with `git diff` before committing
- Check that Markdown renders cleanly in GitHub preview
- Verify symlinks are not broken: `find . -xtype l` (should return nothing)
- When adding/renaming agents or prompts, update indexes:
  - `README.md` - Skills by runtime section
  - `AGENTS.md` - Project structure and Codex CLI Agents sections

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

## Codex CLI Agents

Specialized Claude Code agents that integrate OpenAI Codex CLI capabilities for autonomous development tasks.

### Available Agents

**codex-ask** - Answer questions about code (read-only)

- Uses Codex to analyze code and provide detailed answers
- Includes file references and line numbers
- Shows code examples
- Never modifies code

Use for: Understanding existing code, exploring architecture, finding implementations, learning patterns, debugging assistance

**codex-exec** - Execute development tasks with code modifications

- Generates new code and refactors existing code
- Adds features and fixes bugs
- Creates tests
- Modifies code files

Use for: Code generation, refactoring, feature implementation, bug fixes, test creation

**codex-review** - Perform comprehensive code reviews (read-only)

- Identifies bugs and security vulnerabilities
- Detects performance problems
- Suggests improvements
- Provides actionable feedback
- Never modifies code

Use for: Pre-commit checks, pull request reviews, security audits, performance analysis, code quality assessment

**codex-search** - Search the web for current information (read-only)

- Searches documentation, best practices, and solutions
- Finds API references and tutorials
- Researches library comparisons and approaches
- Provides sourced, verified information
- Never modifies code

Use for: Finding documentation, researching solutions, comparing libraries, learning new technologies, troubleshooting errors

### Usage Patterns

Simply describe your task to Claude Code, and it will launch the appropriate agent:

```
"Can you help me understand how authentication works?"
→ Launches codex-ask agent

"Add input validation to the registration form"
→ Launches codex-exec agent

"Review my changes for security issues"
→ Launches codex-review agent

"What's the best library for JWT authentication in 2026?"
→ Launches codex-search agent
```

### Workflow Patterns

**Understand → Execute → Review**

1. Launch codex-ask: Understand current implementation
2. Launch codex-exec: Make improvements
3. Launch codex-review: Verify changes

**Pre-Commit Workflow**

1. Make changes to code
2. Launch codex-review: Review uncommitted changes
3. Launch codex-exec: Fix identified issues
4. Launch codex-review: Final verification
5. Commit with confidence

**Feature Development**

1. Launch codex-search: Research best practices and libraries
2. Launch codex-ask: Understand existing patterns in codebase
3. Launch codex-exec: Implement following patterns
4. Launch codex-exec: Create tests
5. Launch codex-review: Review implementation

**Research → Understand → Execute → Review**

1. Launch codex-search: Find current documentation and approaches
2. Launch codex-ask: Understand how it fits with current code
3. Launch codex-exec: Implement the solution
4. Launch codex-review: Verify quality and security

### Prerequisites

- Codex CLI installed and available in PATH
- ChatGPT Plus/Pro/Team/Enterprise subscription or OpenAI API key in `~/.codex/config.toml`
- Internet connection for API access

### Configuration

Global config (`~/.codex/config.toml`):

```toml
[general]
model = "gpt-4o"

[execution]
auto_approve = false

[api]
# If using API key:
# key = "sk-..."
```

Project config (`.codex/config.toml`, optional):

```toml
[project]
name = "your-project-name"
language = "typescript"
```

### Agent Files

```
.claude/agents/
├── codex-ask.md     # Question-answering agent
├── codex-exec.md    # Execution agent
├── codex-review.md  # Code review agent
└── codex-search.md  # Web search agent
```

## Spec Kit Workflow

Comprehensive guide to Spec-Driven Development using Spec Kit skills.

### What is Spec-Driven Development?

Spec-Driven Development emphasizes:

- **Intent-driven development** - Define requirements before implementation
- **Multi-step refinement** - Structured progression from idea to implementation
- **Technology independence** - Works with any language or framework
- **AI-assisted execution** - Specifications guide AI coding assistants

### Core Workflow Sequence

The canonical Spec Kit workflow follows these phases in order:

```
1. Constitution (Principles) →
2. Specify (What/Why) →
3. Clarify (Optional) →
4. Plan (How) →
5. Analyze (Optional) →
6. Tasks (Steps) →
7. Implement (Build)
```

### Phase-by-Phase Summary

| Phase | Command                 | Purpose                                  | Output Files                                    | Required        |
| ----- | ----------------------- | ---------------------------------------- | ----------------------------------------------- | --------------- |
| 1     | `/speckit.constitution` | Define project principles and governance | `.specify/memory/constitution.md`               | First time only |
| 2     | `/speckit.specify`      | Capture feature requirements             | `specs/N-name/spec.md`, branch `N-name`         | Yes             |
| 3     | `/speckit.clarify`      | Resolve specification ambiguities        | Updated `spec.md`                               | Optional        |
| 4     | `/speckit.plan`         | Create technical implementation strategy | `plan.md`, `research.md`, `data-model.md`, etc. | Yes             |
| 5     | `/speckit.analyze`      | Validate cross-artifact consistency      | Analysis report                                 | Optional        |
| 6     | `/speckit.tasks`        | Break work into actionable units         | `specs/N-name/tasks.md`                         | Yes             |
| 7     | `/speckit.implement`    | Execute all tasks to build feature       | Implementation code/files                       | Yes             |

**Additional Commands:**

| Command                  | Purpose                               | When to Use                    |
| ------------------------ | ------------------------------------- | ------------------------------ |
| `/speckit.checklist`     | Generate custom validation checklists | Quality assurance at any phase |
| `/speckit.taskstoissues` | Convert tasks to GitHub issues        | Project management integration |

### Command Dependencies

Critical dependencies:

- Constitution must exist before first specify
- Specify must complete before plan
- Plan must complete before tasks
- Tasks must complete before implement
- Clarify can run after specify, before plan
- Analyze can run after tasks, before implement
- Checklist can run at any phase for quality validation

### File Structure After Full Workflow

```
.specify/
├── memory/
│   └── constitution.md           # Project principles (Phase 1)
└── templates/
    ├── spec-template.md
    ├── plan-template.md
    └── tasks-template.md

specs/
└── N-feature-name/               # Feature directory
    ├── spec.md                   # Requirements (Phase 2)
    ├── plan.md                   # Technical plan (Phase 4)
    ├── research.md               # Technical research (Phase 4)
    ├── data-model.md             # Entity definitions (Phase 4)
    ├── tasks.md                  # Work breakdown (Phase 6)
    ├── quickstart.md             # Implementation guide (Phase 4)
    ├── contracts/                # API schemas (Phase 4)
    │   ├── api.yaml
    │   └── schema.graphql
    └── checklists/               # Quality validation
        └── requirements.md       # Spec validation (Phase 2)

src/                              # Implementation (Phase 7)
├── [feature-code]/
└── tests/
```

### Workflow Scenarios

#### Greenfield Development (0-to-1)

Building a new project from scratch:

1. Run `/speckit.constitution` (project setup, once)
2. For each feature:
   - `/speckit.specify` - Create spec
   - `/speckit.clarify` - Optional, resolve ambiguities
   - `/speckit.plan` - Design implementation
   - `/speckit.tasks` - Break down work
   - `/speckit.implement` - Build feature

#### Brownfield Enhancement (Adding to Existing Code)

Adding features to existing codebase:

1. `/speckit.constitution` - Define principles if not already done
2. `/speckit.specify` - Feature spec with integration notes
3. `/speckit.plan` - Integration strategy considering existing architecture
4. `/speckit.analyze` - Validate compatibility
5. `/speckit.tasks` - Incremental changes
6. `/speckit.implement` - Update existing + add new files

#### Creative Exploration (Parallel Approaches)

Testing multiple technical approaches:

1. `/speckit.specify` - Single feature spec
2. `/speckit.plan` - Multiple times with different tech stacks
3. Compare research and contracts from each approach
4. Choose best approach
5. `/speckit.tasks` and `/speckit.implement` - Build selected approach

### Best Practices

**Do's:**

- Follow the sequence - Each phase builds on the previous
- Constitution first - Establish principles before first feature
- Clarify early - Resolve ambiguities before planning
- Validate often - Use analyze and checklist commands
- Document assumptions - Make informed guesses, note them in spec
- Prioritize stories - Use P1/P2/P3 for user stories
- Keep specs technology-agnostic - No implementation details in spec.md

**Don'ts:**

- Don't skip specify - Even small features benefit from structured specs
- Don't put tech in specs - Framework choices belong in plan, not spec
- Don't create circular dependencies - Tasks must be orderable
- Don't ignore constitution - All plans must validate against principles
- Don't over-clarify - Maximum 3 `[NEEDS CLARIFICATION]` markers
- Don't skip tests - Constitution typically mandates test-first

### Troubleshooting

**"No constitution found"**
Solution: Run `/speckit.constitution` first (project initialization).

**"Spec has too many [NEEDS CLARIFICATION] markers"**
Solution: Make informed guesses for non-critical items, document assumptions. Only flag critical decisions.

**"Plan violates constitution principles"**
Solution: Revise plan to align with constitution or justify exception with explicit rationale.

**"Tasks have circular dependencies"**
Solution: Review task dependencies, break circular chains by introducing intermediate tasks.

**"Implementation doesn't match spec"**
Solution: Validate spec and plan first with `/speckit.analyze` before implementing.

### Integration with AI Coding Assistants

Spec Kit works with multiple AI agents:

- **Claude Code** - Uses `.claude/skills/speckit-*` and `.claude/commands/speckit.*`
- **Codex CLI** - Uses `.codex/skills/speckit-*` and `.codex/prompts/speckit.*`
- **GitHub Copilot** - Via custom slash commands
- **Others** - Cursor, Windsurf, Amazon Q, Gemini, etc.

Each agent runtime may have slight variations in invocation, but the workflow sequence remains consistent.
