---
name: speckit-tasks
description: Generate an actionable, dependency-ordered tasks.md for the feature based on available design artifacts.
allowed-tools: Bash, Read, Write, Grep, Glob
handoffs:
  - label: Analyze For Consistency
    agent: speckit.analyze
    prompt: Run a project analysis for consistency
    send: true
  - label: Implement Project
    agent: speckit.implement
    prompt: Start the implementation in phases
    send: true
---

# Spec-Kit Tasks

Generate actionable, dependency-ordered tasks from design artifacts. Third step in spec-kit workflow.

## When to Use

- After creating plan.md with `speckit-plan`
- Ready to break down implementation into tasks
- Need to estimate and organize work

## Execution Workflow

1. **Setup**: Run `.specify/scripts/bash/check-prerequisites.sh --json` to get FEATURE_DIR and AVAILABLE_DOCS
2. **Load design documents**:
   - **Required**: plan.md (tech stack, structure), spec.md (user stories with priorities)
   - **Optional**: data-model.md, contracts/, research.md, quickstart.md
3. **Execute task generation workflow**:
   - Extract tech stack, libraries, project structure from plan.md
   - Extract user stories with priorities (P1, P2, P3) from spec.md
   - Map entities (from data-model.md) and endpoints (from contracts/) to user stories
   - Generate tasks organized by user story in priority order
   - Create dependency graph showing user story completion order
   - Mark parallel execution opportunities with `[P]`
   - Validate task completeness (each user story independently testable)
4. **Generate tasks.md** using `.specify/templates/tasks-template.md` structure:
   - Phase 1: Setup (project initialization)
   - Phase 2: Foundational (blocking prerequisites)
   - Phase 3+: One phase per user story (in priority order)
   - Final Phase: Polish & cross-cutting concerns
   - Each task: `- [ ] [TaskID] [P?] [Story?] Description with file path`
5. **Report**: Output path, total task count, task count per user story, parallel opportunities, suggested MVP scope

## Key Points

- **Task format**: `- [ ] [TaskID] [P?] [Story?] Description with file path`
  - Checkbox required: `- [ ]`
  - Task ID: Sequential (T001, T002, T003...)
  - `[P]` marker: Only if parallelizable
  - `[Story]` label: Required for user story phases ([US1], [US2], etc.)
  - Description must include exact file path
- **Tests are OPTIONAL**: Only generate test tasks if explicitly requested
- **Organization by user story** enables independent implementation and testing
- Each user story phase should be complete and independently testable
- Map all components to their story (models, services, endpoints, tests)
- Break into small, focused tasks (2-8 hours each)
- Mark parallelizable tasks (different files, no dependencies)

## Next Steps

After creating tasks.md:

- **Analyze** for consistency with `speckit-analyze`
- **Start implementation** with `speckit-implement`
- **Convert to issues** with `speckit-taskstoissues` (optional)
