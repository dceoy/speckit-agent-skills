---
name: spec-kit-implement
description: Execute implementation plan by processing and completing all tasks from tasks.md in phases. Use when ready to implement the feature, after creating tasks.md. Fourth step in spec-kit workflow.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Spec-Kit Implement

Execute the implementation by processing tasks from tasks.md in phases. This is the **fourth step** (implementation) in spec-kit workflow.

## When to Use

- After creating tasks.md with `spec-kit-tasks`
- Ready to start implementation
- User asks "let's build this" or "implement the feature"
- Want to execute tasks systematically

## What It Does

1. Loads tasks.md and validates structure
2. Executes tasks phase by phase
3. Marks tasks as completed ([X]) when done
4. Validates checklists (if any)
5. Runs tests after each phase
6. Reports progress

## Prerequisites

- tasks.md must exist (created by `spec-kit-tasks`)
- All design artifacts available (spec.md, plan.md)
- Development environment ready
- Tests configured (if using TDD)

## Basic Usage

### Step 1: Load Tasks

Check prerequisites:

```bash
.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
```

Provides:

- FEATURE_DIR path
- AVAILABLE_DOCS list
- tasks.md content

### Step 2: Validate Checklists (if exist)

Check FEATURE_DIR/checklists/:

```markdown
| Checklist | Total | Completed | Incomplete | Status |
| --------- | ----- | --------- | ---------- | ------ |
| ux.md     | 12    | 12        | 0          | ✓ PASS |
| test.md   | 8     | 5         | 3          | ✗ FAIL |
```

### Step 3: Execute Tasks by Phase

Process each phase sequentially:

**Phase 1: Setup**

- [ ] T001 Create project structure
- [ ] T002 Install dependencies
      → Execute setup tasks
      → Mark completed: [X]

**Phase 2: Foundation**

- [ ] T004 Create User model
- [ ] T005 Create database migration
      → Execute foundational tasks
      → Run tests
      → Mark completed: [X]

**Phase 3+: User Stories**

- Implement one story at a time
- Run story-specific tests
- Mark tasks completed
- Verify acceptance criteria

### Step 4: Verify Each Phase

After each phase, run your project's quality checks:

```bash
<lint-command>    # Check code quality
<test-command>    # Run tests
<build-command>   # Ensure it builds
```

### Step 5: Update tasks.md

Mark completed tasks:

```markdown
- [x] T001 Create project structure
- [x] T002 [P] Install dependencies
- [ ] T003 Configure TypeScript
```

## Implementation Workflow

### Phase Execution

For each phase:

1. **Read all tasks** in the phase
2. **Execute tasks** one by one (or in parallel if marked [P])
3. **Verify completion**:
   - Code works as expected
   - Tests pass
   - No linter errors
4. **Mark as done**: Change `- [ ]` to `- [X]`
5. **Commit**: One commit per task or per phase

### Task Execution Pattern

For each task:

```markdown
Task: T012 [P] [US1] Create registration endpoint in src/api/auth.ts

1. Read: Understand what's needed
2. Implement: Write the code
3. Test: Verify it works
4. Mark done: [X] T012
```

## Best Practices

✅ **DO:**

- Execute tasks in order (respect dependencies)
- Run tests after each phase
- Commit frequently (per task or per phase)
- Mark tasks as done immediately
- Verify acceptance criteria

❌ **DON'T:**

- Skip foundational tasks
- Work on multiple stories simultaneously
- Forget to run tests
- Leave tasks unmarked
- Ignore checklist failures

## Example Workflow

```
tasks.md has 30 tasks across 5 phases

Phase 1: Setup (T001-T003)
→ Execute T001: Create structure
→ Execute T002, T003 in parallel (both have [P])
→ Mark [X] T001-T003
→ Commit: "Setup project structure and dependencies"

Phase 2: Foundation (T004-T006)
→ Execute T004: User model
→ Execute T005, T006 in parallel
→ Run tests
→ Mark [X] T004-T006
→ Commit: "Add foundational models and utilities"

Phase 3: US1 - Registration (T007-T014)
→ Execute T007-T011: Implementation
→ Execute T012-T014: Tests
→ Verify acceptance criteria from spec.md
→ Mark [X] T007-T014
→ Commit: "Implement user registration (US1)"

[Continue for remaining phases...]

Final:
→ All tasks marked [X]
→ All tests passing
→ Feature complete
```

## Checklist Validation

If checklists exist in FEATURE_DIR/checklists/:

```markdown
### Checklist Status (Pre-Implementation)

| Checklist | Total | Completed | Status |
| --------- | ----- | --------- | ------ |
| ux.md     | 12    | 0         | ✗ TODO |
| test.md   | 8     | 0         | ✗ TODO |

### Checklist Status (Post-Implementation)

| Checklist | Total | Completed | Status |
| --------- | ----- | --------- | ------ |
| ux.md     | 12    | 12        | ✓ PASS |
| test.md   | 8     | 8         | ✓ PASS |
```

## Parallel Task Execution

Tasks marked with [P] can run in parallel:

```markdown
Phase 2:

- [ ] T004 Create User model in src/models/user.ts
- [ ] T005 [P] Create Session model in src/models/session.ts
- [ ] T006 [P] Create JWT utils in src/utils/jwt.ts
```

Execute:

- T004 first (no [P])
- T005 and T006 in parallel (both have [P] and different files)

## Testing Strategy

### Per-Phase Testing

After each phase:

```bash
# Run unit tests
<test-command>

# Run integration tests (if applicable)
<integration-test-command>

# Check types (if applicable)
<type-check-command>

# Lint
<lint-command>
```

### User Story Testing

After completing each story phase:

- Verify all acceptance criteria from spec.md
- Run story-specific tests
- Manual testing if needed

## Progress Tracking

Update tasks.md as you go:

```markdown
Progress: 15/30 tasks completed (50%)

Phase 1: Setup ✓ Complete (3/3)
Phase 2: Foundation ✓ Complete (3/3)
Phase 3: US1 🏗️ In Progress (9/14)
Phase 4: US2 ⏳ Pending (0/7)
Phase 5: Polish ⏳ Pending (0/3)
```

## Error Handling

**If task fails:**

1. Understand the issue
2. Fix or adjust approach
3. May need to update plan.md or tasks.md
4. Continue when resolved

**If tests fail:**

1. Fix the failing tests
2. Don't mark task as done
3. Don't proceed to next phase

**If acceptance criteria not met:**

1. Review spec.md requirements
2. Implement missing functionality
3. Verify again

## Commit Strategy

### Option 1: Per Task

```bash
git add .
git commit -m "[T012] Create registration endpoint"
```

### Option 2: Per Phase

```bash
git add .
git commit -m "Phase 3: Implement user registration (US1)

- Created registration endpoint
- Added email validation
- Implemented password hashing
- Generated JWT tokens

All tests passing. US1 acceptance criteria met."
```

## Next Steps

After implementation:

- **Analyze** with `spec-kit-analyze` for consistency
- Manual testing and QA
- Code review
- Deployment planning

## Related Skills

- **spec-kit-tasks** - Creates tasks.md (prerequisite)
- **spec-kit-analyze** - Validates implementation consistency
- **spec-kit-checklist** - Ensures quality standards met

## Common Issues

**Missing tasks.md:**

```
Run spec-kit-tasks first to generate task list
```

**Checklist failures:**

```
Review failing checklist items
Complete required items before declaring feature done
```

**Tests failing:**

```
Fix tests before marking phase complete
Don't skip to next phase with failing tests
```

## Tips for Successful Implementation

1. **Follow the order**: Respect task dependencies
2. **Test continuously**: Don't accumulate technical debt
3. **Mark progress**: Keep tasks.md updated
4. **Commit often**: One commit per task or phase
5. **Verify acceptance**: Check spec.md criteria after each story

---

**Remember**: Implementation is iterative. Mark tasks done as you complete them, test thoroughly, and commit frequently.
