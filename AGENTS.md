# Agent Repository Guidelines

Guidelines for editing agent definitions, skills, and prompts in this repository.

## Project Structure

### Skills (root)

- `skills/` - Source skill directories (Spec Kit workflow skills)

### Claude Code Runtime (`.claude/`)

- `.claude/commands/` - Command prompt files for Spec Kit workflow (speckit.\*.md)
- `.claude/skills` - Symlink to `../skills` (shared skills)

### Codex CLI Runtime (`.codex/`)

- `.codex/prompts/` - Prompt files for Spec Kit workflow (speckit.\*.md)
- `.codex/skills` - Symlink to `../skills` (shared skills)

### Gemini CLI Runtime (`.gemini/`)

- `.gemini/commands/` - Command prompt files for Spec Kit workflow (speckit.\*.toml)

### GitHub Copilot CLI Runtime (`.github/`)

- `.github/agents/` - Spec Kit agent definitions for Copilot CLI (speckit.\*.agent.md)
- `.github/prompts/` - Prompt files for Copilot CLI (speckit.\*.prompt.md)
- `.github/skills/` - Symlink to `../skills` (shared skills)
- `.github/workflows/` - CI workflow definitions (ci.yml)

### Spec Kit Framework (`.specify/`)

- `.specify/templates/` - Templates for spec, plan, tasks, checklist, agent files
- `.specify/memory/` - Constitution and other persistent memory
- `.specify/scripts/` - Helper scripts (bash in `.specify/scripts/bash`)

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
- Skill configs live in `SKILL.md` front matter in each skill directory
- Boolean values: `true`/`false` (lowercase)

**Skill Structure**
Each skill directory must contain:

- `SKILL.md` - Documentation and YAML front matter for configuration and metadata

## Symlink Strategy

This repository uses symlinks to share skills across runtimes:

**Source Skills** (`skills/`)

- Primary location for Spec Kit skills (speckit-\*)
- Contains actual skill directories with `SKILL.md` (front matter + documentation)

**Symlinked Skills**

- `.claude/skills` → `../skills`
- `.codex/skills` → `../skills`
- `.github/skills` → `../skills`

**Benefits**

- Single source of truth for shared skills
- Easy maintenance (edit once, works everywhere)
- Reduced duplication

## Script Conventions

- `.specify/scripts/bash/*.sh` are the canonical helpers for skill workflows.
- Run scripts from repo root and parse their `--json` output; returned paths are absolute.
- Scripts are bash-only; use long-form flags (`--json`, `--paths-only`, etc.) and run via bash/WSL on Windows.

**When Adding/Modifying Skills**

1. Add or edit skills in `skills/`.
2. Ensure the runtime symlinks point at `skills/`:
   ```bash
   ln -s ../skills .claude/skills
   ln -s ../skills .codex/skills
   ln -s ../skills .github/skills
   ```
3. Verify symlinks work: `ls -la .codex/skills/`

## Validation

- Review diffs with `git diff` before committing
- Check that Markdown renders cleanly in GitHub preview
- Verify symlinks are not broken: `find . -xtype l` (should return nothing)
- When adding/renaming agents or prompts, update indexes:
  - `README.md` - Skills by runtime section
  - `AGENTS.md` - Project structure and CLI Agents sections

## Automation

- `.github/workflows/ci.yml` runs `specify init --force --here --ai` across runtimes, formats Markdown/JSON outputs, and regenerates `VERSION.md` from `specify version`. Avoid hand-editing generated assets when rerunning the workflow locally.

## Commit & Pull Request Guidelines

- Commit messages are short, imperative, sentence-case.
- Branch names use appropriate prefixes on creation (e.g., `feature/short-description`, `bugfix/short-description`).
- PRs should include: a clear summary, relevant context or linked issue.
- When instructed to create a PR, create it as a draft with appropriate labels by default.

## Agent-Specific Notes

- Agent and prompt files are the product of this repo—treat them as source of truth.
- Prefer small, focused edits per file for reviewability.

## Runtime Assets

- **Skills**: All skills live in `skills/` (speckit-\* workflow skills) and are shared via symlinks in `.claude/skills`, `.codex/skills`, and `.github/skills`.
- **Claude Code**: Spec Kit command prompts in `.claude/commands/`; uses shared skills via the symlinked `.claude/skills`.
- **Codex CLI**: Spec Kit prompt files in `.codex/prompts/`; uses shared skills via the symlinked `.codex/skills`.
- **GitHub Copilot CLI**: Spec Kit agents in `.github/agents/`, prompts in `.github/prompts/`, and shared skills via `.github/skills`.
- **Gemini CLI**: Spec Kit command prompts in `.gemini/commands/`.

### Current skill inventory

- All runtimes include: `speckit-constitution`, `speckit-specify`, `speckit-baseline`, `speckit-clarify`, `speckit-plan`, `speckit-analyze`, `speckit-tasks`, `speckit-implement`, `speckit-checklist`, `speckit-taskstoissues`.
- File patterns by runtime:
  - Source configs: `skills/speckit-*/SKILL.md`
  - Claude commands: `.claude/commands/speckit.<name>.md`
  - Codex prompts: `.codex/prompts/speckit.<name>.md`
  - Copilot prompts/agents: `.github/prompts/speckit.<name>.prompt.md` and `.github/agents/speckit.<name>.agent.md`
  - Gemini commands: `.gemini/commands/speckit.<name>.toml`

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
5. Tasks (Steps) →
6. Analyze (Optional) →
7. Implement (Build)
```

### Phase-by-Phase Summary

| Phase | Agent skill            | Purpose                                  | Output Files                                    | Required        |
| ----- | ---------------------- | ---------------------------------------- | ----------------------------------------------- | --------------- |
| 1     | `speckit-constitution` | Define project principles and governance | `.specify/memory/constitution.md`               | First time only |
| 2     | `speckit-specify`      | Capture feature requirements             | `specs/N-name/spec.md`, branch `N-name`         | Yes             |
| 3     | `speckit-clarify`      | Resolve specification ambiguities        | Updated `spec.md`                               | Optional        |
| 4     | `speckit-plan`         | Create technical implementation strategy | `plan.md`, `research.md`, `data-model.md`, etc. | Yes             |
| 5     | `speckit-tasks`        | Break work into actionable units         | `specs/N-name/tasks.md`                         | Yes             |
| 6     | `speckit-analyze`      | Validate cross-artifact consistency      | Analysis report                                 | Optional        |
| 7     | `speckit-implement`    | Execute all tasks to build feature       | Implementation code/files                       | Yes             |

**Additional Agent skills:**

| Agent skill             | Purpose                               | When to Use                      |
| ----------------------- | ------------------------------------- | -------------------------------- |
| `speckit-baseline`      | Generate specs from existing code     | Documenting legacy/existing code |
| `speckit-checklist`     | Generate custom validation checklists | Quality assurance at any phase   |
| `speckit-taskstoissues` | Convert tasks to GitHub issues        | Project management integration   |

### Skill Dependencies

Critical dependencies:

- Constitution must exist before first specify or baseline
- Specify or baseline must complete before clarify (if used) and before plan
- Plan must complete before tasks
- Tasks must complete before analyze, implement, or taskstoissues
- Analyze is optional but runs after tasks and before implement
- Checklist can run at any phase for quality validation
- Baseline can be used instead of specify when documenting existing code

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
    ├── tasks.md                  # Work breakdown (Phase 5)
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

1. Run the `speckit-constitution` agent skill (project setup, once)
2. For each feature:
   - `speckit-specify` - Create spec
   - `speckit-clarify` - Optional, resolve ambiguities
   - `speckit-plan` - Design implementation
   - `speckit-tasks` - Break down work
   - `speckit-implement` - Build feature

#### Brownfield Enhancement (Adding to Existing Code)

Adding features to existing codebase:

1. `speckit-constitution` - Define principles if not already done
2. `speckit-baseline src/existing-feature/` - Generate specs from existing code (optional, for documentation)
3. `speckit-specify` - Feature spec with integration notes
4. `speckit-plan` - Integration strategy considering existing architecture
5. `speckit-tasks` - Incremental changes
6. `speckit-analyze` - Validate compatibility
7. `speckit-implement` - Update existing + add new files

#### Legacy Code Documentation

Documenting existing code before refactoring:

1. `speckit-constitution` - Define principles if not already done
2. `speckit-baseline src/legacy-module/` - Generate specs from code
3. `speckit-clarify` - Review and validate generated requirements
4. `speckit-plan` - Plan modernization or refactoring
5. `speckit-tasks` - Break down changes
6. `speckit-implement` - Execute refactoring

#### Creative Exploration (Parallel Approaches)

Testing multiple technical approaches:

1. `speckit-specify` - Single feature spec
2. `speckit-plan` - Multiple times with different tech stacks
3. Compare research and contracts from each approach
4. Choose best approach
5. `speckit-tasks` and `speckit-implement` - Build selected approach

### Best Practices

**Do's:**

- Follow the sequence - Each phase builds on the previous
- Constitution first - Establish principles before first feature
- Clarify early - Resolve ambiguities before planning
- Validate often - Use analyze and checklist skills
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
Solution: Run the `speckit-constitution` agent skill first (project initialization).

**"Spec has too many [NEEDS CLARIFICATION] markers"**
Solution: Make informed guesses for non-critical items, document assumptions. Only flag critical decisions.

**"Plan violates constitution principles"**
Solution: Revise plan to align with constitution or justify exception with explicit rationale.

**"Tasks have circular dependencies"**
Solution: Review task dependencies, break circular chains by introducing intermediate tasks.

**"Implementation doesn't match spec"**
Solution: Validate spec, plan, and tasks first with `speckit-analyze` before implementing.

### Integration with AI Coding Assistants

Spec Kit works with multiple AI agents:

- **Claude Code** - Uses `.claude/skills/speckit-*`
- **Codex CLI** - Uses `.codex/skills/speckit-*`
- **GitHub Copilot** - Uses `.github/skills/speckit-*`

Each agent runtime may have slight variations in invocation, but the workflow sequence remains consistent.
