---
name: gh-issue-develop
description: Start working on a GitHub issue by creating a linked branch using gh CLI. Automatically creates and checks out a branch linked to the issue. Use when beginning work on an issue.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: Create PR
    agent: gh-pr-create
    prompt: Create a pull request for this branch
    send: true
  - label: View Issue
    agent: gh-issue-view
    prompt: View the issue details
    send: true
  - label: Update Issue
    agent: gh-issue-comment
    prompt: Add progress comment to the issue
    send: true
---

# GitHub Issue Develop Skill

Start working on GitHub issues by creating linked branches using the `gh` CLI.

## When to Use

- User says "start working on issue #123" or "create branch for issue"
- Beginning development on an issue
- Need to link work to specific issue
- Want automatic branch naming from issue
- Setting up development environment for issue

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

Verify git repository:

```bash
git status
```

## Execution Workflow

### Step 1: Verify Issue Exists

Check the issue before creating branch:

```bash
# View issue
gh issue view 123 --json number,title,state,assignees

# Ensure issue is open
STATE=$(gh issue view 123 --json state --jq '.state')
if [ "$STATE" != "OPEN" ]; then
  echo "⚠️ Issue #123 is closed"
  exit 1
fi
```

### Step 2: Check Current State

Ensure clean working directory:

```bash
# Check for uncommitted changes
if ! git diff-index --quiet HEAD --; then
  echo "⚠️ You have uncommitted changes. Commit or stash them first."
  exit 1
fi

# Check current branch
CURRENT_BRANCH=$(git branch --show-current)
echo "Currently on branch: $CURRENT_BRANCH"
```

### Step 3: Create and Checkout Branch

**Basic usage:**

```bash
# Create branch linked to issue
gh issue develop 123 --checkout
```

**With custom branch name:**

```bash
# Specify branch name
gh issue develop 123 --name "fix/login-safari-bug" --checkout
```

**Without automatic checkout:**

```bash
# Create but don't checkout
gh issue develop 123
```

**From specific base branch:**

```bash
# Create from develop instead of main
gh issue develop 123 --base develop --checkout
```

### Step 4: Verify Branch Creation

Confirm branch was created and linked:

```bash
# Check current branch
git branch --show-current

# View branch details
git log -1 --oneline

# Verify issue link (check branch name or description)
gh api repos/:owner/:repo/issues/123/timeline \
  | jq -r '.[] | select(.event == "referenced")'
```

### Step 5: Update Issue Status

Add comment that work has started:

```bash
gh issue comment 123 --body "$(cat <<'EOF'
Started working on this issue.

Branch: `$(git branch --show-current)`

Next steps:
- Reproduce the issue locally
- Implement fix
- Add tests
- Create PR

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

# Optionally add "in-progress" label
gh issue edit 123 --add-label "in-progress"
```

### Step 6: Report to User

```markdown
✓ Ready to work on issue #123

Branch: fix/login-safari-bug
Base: main
Status: Checked out and ready

Issue: Fix login button not responding on Safari
Link: https://github.com/owner/repo/issues/123

Next steps:

- Make your changes
- Commit your work
- Create PR with `gh-pr-create` skill
```

## Common Scenarios

### Scenario 1: Start Work on Bug

```bash
# Start working on bug report
gh issue develop 123 --checkout

# Add in-progress label
gh issue edit 123 --add-label "in-progress"

# Comment on issue
gh issue comment 123 --body "🔨 Started working on this bug"

# Begin work
echo "Ready to fix the bug!"
```

### Scenario 2: Custom Branch Naming

```bash
# Get issue title for branch name
ISSUE_TITLE=$(gh issue view 123 --json title --jq '.title')
BRANCH_NAME=$(echo "$ISSUE_TITLE" | \
  tr '[:upper:]' '[:lower:]' | \
  tr ' ' '-' | \
  sed 's/[^a-z0-9-]//g' | \
  cut -c1-50)

# Create branch with custom name
gh issue develop 123 \
  --name "feature/$BRANCH_NAME" \
  --checkout

echo "Created branch: feature/$BRANCH_NAME"
```

### Scenario 3: Start Multiple Related Issues

```bash
# Work on related issues sequentially
for issue in 123 124 125; do
  echo "Starting work on issue #$issue"

  # Create branch
  gh issue develop $issue --checkout

  # Add label
  gh issue edit $issue --add-label "in-progress"

  # Prompt user to work on it
  echo "Work on issue #$issue now. Press Enter when done..."
  read

  # Commit work
  git add .
  git commit -m "Work on issue #$issue"

  # Go back to main
  git checkout main
done
```

### Scenario 4: Create Branch from Different Base

```bash
# Create feature branch from develop
gh issue develop 123 \
  --base develop \
  --name "feature/new-export-format" \
  --checkout

# Verify base
git log --oneline origin/develop..HEAD
```

### Scenario 5: Create Branch Without Checkout

```bash
# Create branch but stay on current branch
gh issue develop 123 --name "fix/issue-123"

# Work on other things first
git add .
git commit -m "Finish current task"

# Now switch to issue branch
git checkout fix/issue-123
```

### Scenario 6: Assign and Start Work

```bash
# Assign to yourself and start working
gh issue edit 123 --add-assignee "@me"
gh issue develop 123 --checkout
gh issue edit 123 --add-label "in-progress"

gh issue comment 123 --body "Assigned to myself and starting work now."
```

### Scenario 7: Full Workflow Start to Finish

```bash
# Complete workflow for starting issue work
ISSUE_NUM=123

echo "=== Starting work on issue #$ISSUE_NUM ==="

# 1. View issue
gh issue view $ISSUE_NUM

# 2. Assign to self
gh issue edit $ISSUE_NUM --add-assignee "@me"

# 3. Create and checkout branch
gh issue develop $ISSUE_NUM --checkout

# 4. Update status
gh issue edit $ISSUE_NUM --add-label "in-progress"

# 5. Add comment
BRANCH=$(git branch --show-current)
gh issue comment $ISSUE_NUM --body "Started work in branch \`$BRANCH\`"

# 6. Ready to code
echo "✓ Ready to work! Branch: $BRANCH"
```

## Advanced Usage

### Auto-generate Branch Name from Issue

```bash
# Create branch with issue number prefix
gh issue develop 123 --checkout

# Resulting branch might be: issue-123 or 123-fix-login-bug
```

### Link Existing Branch to Issue

```bash
# If you already created a branch manually
git checkout -b fix/login-bug

# Reference issue in first commit
git commit --allow-empty -m "Start work on #123"

# Or add issue reference later
gh issue comment 123 --body "Working on this in branch \`fix/login-bug\`"
```

### Create Branch with Issue Template

```bash
# Create branch and add starter code from template
gh issue develop 123 --checkout

# Copy template files
cp templates/feature.ts src/new-feature.ts

# Add starter commit
git add src/new-feature.ts
git commit -m "Initial implementation for #123

Scaffolding:
- Added feature file
- Set up basic structure
- TODO: Implement core logic

Refs #123"
```

### Development Workflow Integration

```bash
# Full development workflow
start_issue() {
  local issue=$1

  # Validate issue
  if ! gh issue view $issue &>/dev/null; then
    echo "Error: Issue #$issue not found"
    return 1
  fi

  # Create branch
  gh issue develop $issue --checkout || return 1

  # Update issue
  gh issue edit $issue --add-assignee "@me" --add-label "in-progress"

  # Add comment
  gh issue comment $issue --body "🚀 Started development"

  # Setup environment
  npm install
  npm run dev

  echo "✓ Ready to develop issue #$issue"
}

# Usage
start_issue 123
```

## Branch Naming Conventions

**Auto-generated names typically follow:**

```
issue-123
123-short-issue-title
fix-short-issue-title
feature-short-issue-title
```

**Custom naming patterns:**

```
{type}/{issue-number}-{description}

Examples:
- feature/123-export-csv
- fix/123-login-safari
- refactor/123-auth-module
- docs/123-api-guide
```

**Type prefixes:**

- `feature/` - New features
- `fix/` - Bug fixes
- `refactor/` - Code refactoring
- `docs/` - Documentation
- `test/` - Test additions
- `chore/` - Maintenance tasks

## Tips

- **Clean working directory**: Always commit or stash before creating branch
- **Descriptive names**: Use clear branch names that indicate purpose
- **Update issue**: Add "in-progress" label when starting work
- **Comment early**: Let stakeholders know you're working on it
- **Link in commits**: Reference issue number in commit messages
- **Base branch matters**: Create from correct base (main/develop/etc)
- **Delete when done**: Clean up branches after PR is merged

## Error Handling

**Error: "Issue not found"**

- Cause: Issue doesn't exist or no access
- Solution: Verify issue number with `gh issue list`

**Error: "Not authorized"**

- Cause: Not authenticated or no repository access
- Solution: Run `gh auth login`

**Error: "Branch already exists"**

- Cause: Branch with that name exists
- Solution: Use different name with `--name` flag or delete old branch

**Error: "Uncommitted changes"**

- Cause: Working directory not clean
- Solution: Commit or stash changes first

**Error: "Base branch not found"**

- Cause: Specified base branch doesn't exist
- Solution: Verify base branch name with `git branch -a`

## Best Practices

1. **Start with clean state**: Commit or stash before creating branch
2. **Use clear branch names**: Make purpose obvious from name
3. **Update issue immediately**: Add in-progress label and comment
4. **Work on one issue at a time**: Focus on single issue per branch
5. **Commit often**: Make small, focused commits
6. **Reference issue**: Use "#123" in commit messages
7. **Keep branches short-lived**: Merge PRs quickly
8. **Delete merged branches**: Clean up after PR is merged
9. **Sync with base**: Regularly update from main/develop
10. **Document assumptions**: Note any decisions in issue comments

## Workflow Checklist

```markdown
Before starting work:

- [ ] Issue is assigned to you
- [ ] Issue requirements are clear
- [ ] Working directory is clean
- [ ] On correct base branch

When starting work:

- [ ] Create linked branch with `gh issue develop`
- [ ] Add "in-progress" label
- [ ] Comment on issue that work started
- [ ] Reference issue in commits

When finishing work:

- [ ] All tests pass
- [ ] Code reviewed locally
- [ ] Create PR linked to issue
- [ ] Request reviews
- [ ] Update issue with PR link
```

## Related Skills

- `gh-issue-view` - View issue before starting
- `gh-issue-edit` - Update issue metadata
- `gh-issue-comment` - Add progress updates
- `gh-pr-create` - Create PR when work is complete

## Limitations

- Requires write access to repository
- Cannot create branch if name already exists
- Branch must be created from existing base branch
- Some repositories may have branch protection rules
- Feature may not be available in older gh CLI versions

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_develop
- Branch workflow: https://docs.github.com/en/get-started/quickstart/github-flow
- Linking issues: https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue
