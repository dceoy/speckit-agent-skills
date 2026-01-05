---
name: gh-pr-ready
description: Mark a draft pull request as ready for review using gh CLI. Use when a draft PR is complete and ready for team review.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: View PR
    agent: gh-pr-view
    prompt: View the PR details
    send: true
  - label: Request Reviews
    agent: gh-pr-review
    prompt: Request reviews from team members
    send: true
---

# GitHub PR Ready Skill

Convert draft pull requests to ready-for-review status using the `gh` CLI.

## When to Use

- User says "mark PR as ready" or "ready for review"
- Draft PR is complete and tests pass
- User wants to notify reviewers
- After addressing WIP/TODO items in draft PR

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

## Execution Workflow

### Step 1: Identify the PR

**If user specifies PR number:**

```bash
gh pr view 123 --json number,isDraft,title
```

**If on a branch with PR:**

```bash
gh pr view --json number,isDraft,title,url
```

**If user doesn't specify (find draft PRs):**

```bash
# List all draft PRs by current user
gh pr list --author "@me" --state open --json number,title,isDraft \
  | jq '.[] | select(.isDraft == true)'
```

### Step 2: Verify Draft Status

Check if the PR is actually a draft:

```bash
IS_DRAFT=$(gh pr view 123 --json isDraft --jq '.isDraft')

if [ "$IS_DRAFT" != "true" ]; then
  echo "PR #123 is already ready for review"
  exit 0
fi
```

### Step 3: Pre-Flight Checks

Before marking ready, verify:

1. **All commits are pushed:**

```bash
git status
git log origin/main..HEAD
```

2. **CI checks status:**

```bash
gh pr checks 123
```

3. **No merge conflicts:**

```bash
gh pr view 123 --json mergeable --jq '.mergeable'
```

Report any issues to user before proceeding.

### Step 4: Mark as Ready

```bash
gh pr ready 123
```

Or for current branch:

```bash
gh pr ready
```

### Step 5: Optional - Request Reviews

After marking ready, optionally request reviews:

```bash
# Request specific reviewers
gh pr edit 123 --add-reviewer user1,user2,team/backend

# Or request reviews from CODEOWNERS
gh pr edit 123 --add-reviewer $(gh api repos/:owner/:repo/contents/.github/CODEOWNERS --jq '.content' | base64 -d | grep -v '^#' | awk '{print $2}' | paste -sd,)
```

### Step 6: Notify and Report

Present the result:

- PR number and URL
- New status (Ready for review)
- Reviewers (if added)
- CI check status
- Next steps

## Common Scenarios

### Scenario 1: Simple Ready Conversion

```bash
# On feature branch with draft PR
git status
gh pr view --json number,title,isDraft

# Verify no issues
gh pr checks

# Mark ready
gh pr ready

echo "✓ PR marked as ready for review"
gh pr view --web  # Optional: open in browser
```

### Scenario 2: Mark Ready with Reviewers

```bash
# Mark ready and request reviews in one flow
gh pr ready 123

gh pr edit 123 \
  --add-reviewer user1,user2 \
  --add-label ready-for-review

echo "✓ PR #123 ready and reviewers notified"
```

### Scenario 3: Mark Ready After CI Passes

```bash
# Wait for CI checks
echo "Checking CI status..."
gh pr checks 123 --watch

# Once passed, mark ready
if gh pr checks 123 | grep -q "✓"; then
  gh pr ready 123
  gh pr edit 123 --add-reviewer team/reviewers
  echo "✓ CI passed, PR ready for review"
else
  echo "⚠ CI checks failing. Fix issues before marking ready."
  exit 1
fi
```

### Scenario 4: Bulk Ready Conversion

```bash
# Mark multiple draft PRs as ready
for pr in $(gh pr list --author "@me" --json number,isDraft --jq '.[] | select(.isDraft == true) | .number'); do
  echo "Processing PR #$pr..."
  gh pr checks $pr --watch
  gh pr ready $pr
  echo "✓ PR #$pr marked ready"
done
```

### Scenario 5: Ready with Summary Update

```bash
# Update PR description before marking ready
gh pr edit 123 --body "$(cat <<'EOF'
## Summary
Feature implementation complete and tested.

## Changes
- Implemented user authentication
- Added JWT token handling
- Created login/logout flows

## Testing
- ✓ All unit tests pass
- ✓ Integration tests pass
- ✓ Manual testing completed

## Checklist
- [x] Tests added
- [x] Documentation updated
- [x] No breaking changes
- [x] CI passing

Ready for review!

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"

# Then mark ready
gh pr ready 123
```

## Advanced Options

### Conditional Ready

Only mark ready if certain conditions are met:

```bash
# Check CI status
CI_STATUS=$(gh pr checks 123 --json state --jq '.[].state' | sort -u)

# Check coverage (example)
COVERAGE=$(grep -o 'Coverage: [0-9]*%' coverage/report.txt | grep -o '[0-9]*')

# Mark ready only if CI passes and coverage > 80%
if [[ "$CI_STATUS" == "SUCCESS" ]] && [[ "$COVERAGE" -gt 80 ]]; then
  gh pr ready 123
  echo "✓ PR ready (CI passed, coverage: $COVERAGE%)"
else
  echo "⚠ Not ready: CI=$CI_STATUS, Coverage=$COVERAGE%"
  exit 1
fi
```

### Ready with Auto-merge

Mark ready and enable auto-merge:

```bash
gh pr ready 123
gh pr merge 123 --auto --squash
echo "✓ PR ready with auto-merge enabled"
```

### Team Notification

Mark ready and notify team in Slack/Discord:

```bash
gh pr ready 123

PR_URL=$(gh pr view 123 --json url --jq '.url')
PR_TITLE=$(gh pr view 123 --json title --jq '.title')

# Example Slack webhook notification
curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
  -H 'Content-Type: application/json' \
  -d "{\"text\":\"PR ready for review: $PR_TITLE - $PR_URL\"}"
```

## Tips

- **Check CI first**: Always verify CI passes before marking ready
- **Update description**: Ensure PR body reflects final state
- **Remove WIP markers**: Update title to remove "[WIP]" or "[Draft]"
- **Add reviewers immediately**: Don't wait for automatic CODEOWNERS assignment
- **Use labels**: Add "ready-for-review" label for better visibility
- **Notify team**: Consider notifying team in chat for urgent reviews

## Error Handling

**Error: "PR is not a draft"**

- Cause: PR is already ready for review
- Solution: Verify with `gh pr view 123 --json isDraft`

**Error: "Not found"**

- Cause: PR doesn't exist or you don't have access
- Solution: Check PR number with `gh pr list`

**Error: "CI checks failing"**

- Cause: PR has failing CI checks
- Solution: Fix issues first, or mark ready with warning

**Error: "Merge conflicts"**

- Cause: PR has conflicts with base branch
- Solution: Resolve conflicts first with `git merge origin/main`

**Error: "GraphQL error"**

- Cause: GitHub API issue or rate limit
- Solution: Wait and retry, or check `gh auth status`

## Best Practices

1. **Pre-flight checks**: Always verify CI and tests before marking ready
2. **Update description**: Ensure PR body is current and complete
3. **Remove TODO markers**: Clear all WIP/TODO comments from code
4. **Add reviewers immediately**: Don't rely solely on CODEOWNERS
5. **Clean up commits**: Consider squashing or rebasing before marking ready
6. **Update title**: Remove "[Draft]" or "[WIP]" prefixes
7. **Check dependencies**: Ensure dependent PRs are merged first
8. **Verify no conflicts**: Resolve merge conflicts before review
9. **Documentation**: Ensure all docs are updated
10. **Self-review**: Review your own PR once more before marking ready

## Related Skills

- `gh-pr-create` - Create new pull requests
- `gh-pr-view` - View PR details
- `gh-pr-review` - Review and approve PRs
- `gh-pr-merge` - Merge approved PRs

## Limitations

- Requires GitHub CLI installed and authenticated
- Can only mark your own PRs or PRs you have write access to
- Cannot mark ready if PR has required checks that are failing (depends on repo settings)
- Some organizations may require specific workflows before marking ready

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_pr_ready
- Draft PRs: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests
