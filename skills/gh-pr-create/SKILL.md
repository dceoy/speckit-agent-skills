---
name: gh-pr-create
description: Create a GitHub pull request using gh CLI. Supports standard PRs, draft PRs, auto-merge, and custom reviewers/labels. Use when the user wants to create a PR from their current branch.
allowed-tools: Bash, Read, Grep, Glob
handoffs:
  - label: Mark PR Ready
    agent: gh-pr-ready
    prompt: Mark the draft PR as ready for review
    send: true
  - label: View PR Details
    agent: gh-pr-view
    prompt: View the created PR
    send: true
---

# GitHub PR Create Skill

Create GitHub pull requests using the `gh` CLI with intelligent defaults and comprehensive options.

## When to Use

- User says "create a pull request" or "create a PR"
- User wants to open a PR from current branch
- After completing feature work and wanting to merge
- User asks to create a draft PR for early feedback
- User wants to request reviews from specific people/teams

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

If not authenticated:

```bash
gh auth login
```

## Execution Workflow

### Step 1: Gather Context

Before creating the PR, collect information:

```bash
# Check current branch and status
git status
git branch --show-current

# Check if branch has upstream and is pushed
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "No upstream"

# View commits that will be in PR
git log origin/main..HEAD --oneline

# View diff from base branch
git diff origin/main...HEAD --stat
```

### Step 2: Determine PR Details

**Title**: Use the branch name or most recent commit message as default

- Branch name: `feature/add-auth` → "Add auth"
- Commit message: "Add user authentication system"

**Body**: Generate structured description:

```markdown
## Summary

[Brief description of changes - 1-3 bullet points]

## Changes

- [List of key changes based on commits and diff]

## Testing

[How the changes were tested]

## Related Issues

Closes #[issue-number] (if applicable)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**Base Branch**: Default to `main`, but check repository default:

```bash
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

### Step 3: Push Branch (if needed)

If branch is not pushed or not up to date:

```bash
git push -u origin $(git branch --show-current)
```

### Step 4: Create the PR

**Standard PR:**

```bash
gh pr create \
  --title "PR Title Here" \
  --body "$(cat <<'EOF'
## Summary
- Key change 1
- Key change 2

## Testing
Tested locally with...

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**With Options:**

```bash
gh pr create \
  --title "PR Title" \
  --body "..." \
  --draft \                           # Create as draft
  --base main \                       # Target branch
  --assignee @me \                    # Assign to yourself
  --reviewer user1,user2 \            # Request reviews
  --label bug,priority-high \         # Add labels
  --milestone v1.0 \                  # Set milestone
  --web                               # Open in browser
```

**Auto-merge (if enabled):**

```bash
gh pr create --title "..." --body "..." --assignee @me

# Enable auto-merge after creation
gh pr merge --auto --squash  # or --merge or --rebase
```

### Step 5: Capture PR URL

The `gh pr create` command returns the PR URL. Parse and display it:

```bash
PR_URL=$(gh pr create --title "..." --body "..." 2>&1 | grep -o 'https://github.com/[^[:space:]]*')
echo "Created PR: $PR_URL"
```

### Step 6: Report to User

Present the result with:

- PR number and URL
- Title and summary
- Reviewers (if any)
- Draft status (if applicable)
- Next steps (e.g., "Request reviews", "Mark as ready")

## Common Scenarios

### Scenario 1: Simple Feature PR

```bash
# Context: Feature branch with 3 commits, ready to merge

git status
git log origin/main..HEAD --oneline
git diff origin/main...HEAD --stat

gh pr create \
  --title "Add user profile editing" \
  --body "$(cat <<'EOF'
## Summary
- Adds profile editing UI
- Implements profile update API
- Adds validation for profile fields

## Testing
- Manually tested all form fields
- Added unit tests for validation
- Tested on Chrome and Firefox

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Scenario 2: Draft PR for Early Feedback

```bash
gh pr create \
  --draft \
  --title "[WIP] Refactor authentication system" \
  --body "$(cat <<'EOF'
## Summary
Early draft for feedback on authentication refactor approach.

## Changes So Far
- Extracted auth logic into separate module
- Updated login flow

## TODO
- [ ] Add tests
- [ ] Update documentation
- [ ] Handle edge cases

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Scenario 3: PR with Reviewers and Labels

```bash
gh pr create \
  --title "Fix payment processing bug" \
  --body "..." \
  --reviewer @user1,team/backend-team \
  --label bug,priority-high \
  --assignee @me
```

### Scenario 4: Update Existing Draft

If a draft PR already exists, update it instead:

```bash
# Check if PR exists for current branch
gh pr view --json number,isDraft 2>/dev/null

# Update existing PR
gh pr edit \
  --title "Updated Title" \
  --body "Updated description"
```

## Advanced Options

### Custom Base Branch

For PRs targeting a specific branch:

```bash
gh pr create --base develop --title "..." --body "..."
```

### From Fork

When working on a fork:

```bash
gh pr create \
  --repo upstream-owner/repo \
  --head your-username:feature-branch \
  --base main \
  --title "..." \
  --body "..."
```

### Interactive Mode

Let GitHub CLI prompt for all fields:

```bash
gh pr create
# CLI will interactively ask for title, body, base branch, etc.
```

### Bulk PR Creation

For multiple related branches:

```bash
for branch in feature-1 feature-2 feature-3; do
  git checkout $branch
  gh pr create --title "$(echo $branch | sed 's/-/ /g')" --body "..."
done
```

## Tips

- **Use HEREDOC for body**: Ensures proper formatting and avoids quoting issues
- **Analyze commits**: Use `git log` to understand what changed
- **Check diff stats**: Use `git diff --stat` to see file changes
- **Push before creating**: Ensure branch is pushed with `git push -u origin HEAD`
- **Open in browser**: Add `--web` flag to review PR after creation
- **Save PR URL**: Store in variable for follow-up actions

## Error Handling

**Error: "No commits between base and head"**

- Cause: Branch is already merged or no new commits
- Solution: Check `git log origin/main..HEAD`

**Error: "Pull request already exists"**

- Cause: PR already exists for this branch
- Solution: Use `gh pr view` to see existing PR, or `gh pr edit` to update

**Error: "Not authenticated"**

- Cause: Not logged into GitHub CLI
- Solution: Run `gh auth login`

**Error: "Push rejected"**

- Cause: Branch not pushed or conflicts exist
- Solution: Push with `git push -u origin HEAD`

**Error: "Invalid reviewer"**

- Cause: Reviewer username/team doesn't exist or no permission
- Solution: Verify names with `gh api repos/:owner/:repo/collaborators`

## Best Practices

1. **Always review changes first**: Use `git diff` and `git log` to understand what you're proposing
2. **Write clear titles**: Use imperative mood (e.g., "Add feature" not "Added feature")
3. **Structured bodies**: Use sections like Summary, Changes, Testing, Related Issues
4. **Link issues**: Use "Closes #123" to auto-close issues when PR merges
5. **Use draft for WIP**: Create draft PRs for early feedback, mark ready when complete
6. **Request specific reviewers**: Don't rely on CODEOWNERS alone, explicitly request reviews
7. **Add labels early**: Helps with project management and filtering
8. **Assign yourself**: Makes it clear who owns the PR
9. **Check CI status**: After creation, verify CI checks pass

## Related Skills

- `gh-pr-ready` - Mark draft PR as ready for review
- `gh-pr-view` - View PR details and status
- `gh-pr-merge` - Merge a PR
- `gh-pr-review` - Review and comment on PRs

## Limitations

- Requires GitHub CLI installed and authenticated
- Requires branch to be pushed to remote
- Cannot create PRs for repositories you don't have access to
- Some options (like auto-merge) require repository settings to be enabled

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_pr_create
- Creating PRs: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests
