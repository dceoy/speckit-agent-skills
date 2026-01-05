---
name: gh-pr-merge
description: Merge GitHub pull requests using gh CLI. Supports merge, squash, and rebase strategies with auto-merge option. Use when PR is approved and ready to merge.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: View PR
    agent: gh-pr-view
    prompt: View the merged PR
    send: true
---

# GitHub PR Merge Skill

Merge approved pull requests using the `gh` CLI with intelligent merge strategies.

## When to Use

- User says "merge the PR" or "merge this pull request"
- PR is approved and ready to merge
- User wants to enable auto-merge
- After all CI checks pass and reviews are approved
- User wants to merge with specific strategy (squash/rebase)

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

Verify merge permissions:

```bash
gh repo view --json viewerPermission --jq '.viewerPermission'
# Should be: WRITE, MAINTAIN, or ADMIN
```

## Execution Workflow

### Step 1: Pre-Merge Validation

Before merging, verify the PR is ready:

```bash
# Get PR status
gh pr view 123 --json \
  state,isDraft,mergeable,reviewDecision,\
  statusCheckRollup,commits

# Check if:
# - state == "OPEN"
# - isDraft == false
# - mergeable == "MERGEABLE"
# - reviewDecision == "APPROVED" (if required)
# - statusCheckRollup shows all required checks passed
```

**Validation Checklist:**

- [ ] PR is open (not closed/merged)
- [ ] PR is not a draft
- [ ] No merge conflicts
- [ ] Required reviews approved
- [ ] All required CI checks passed
- [ ] No blocking conversations

### Step 2: Choose Merge Strategy

Determine merge strategy based on:

- Repository settings
- Team conventions
- PR characteristics

**Merge Strategies:**

1. **Merge Commit** (`--merge`):
   - Preserves all commits and history
   - Creates merge commit
   - Use for: Feature branches with meaningful commit history

2. **Squash and Merge** (`--squash`):
   - Combines all commits into one
   - Clean linear history
   - Use for: Feature PRs with many WIP commits

3. **Rebase and Merge** (`--rebase`):
   - Replays commits on base branch
   - No merge commit
   - Use for: Clean commit history, linear timeline

**Auto-detect strategy:**

```bash
# Check repository default merge method
MERGE_METHOD=$(gh api repos/:owner/:repo --jq '
  if .allow_squash_merge and .allow_merge_commit and .allow_rebase_merge then
    "ask"
  elif .allow_squash_merge then
    "squash"
  elif .allow_rebase_merge then
    "rebase"
  elif .allow_merge_commit then
    "merge"
  else
    "error"
  end
')
```

### Step 3: Merge the PR

**Standard merge:**

```bash
gh pr merge 123 --squash
```

**With custom commit message:**

```bash
gh pr merge 123 --squash \
  --subject "Add user authentication" \
  --body "$(cat <<'EOF'
Implements JWT-based authentication with:
- Login/logout endpoints
- Token refresh mechanism
- Password hashing with bcrypt
- Session management

Closes #45
EOF
)"
```

**Auto-merge (merge when checks pass):**

```bash
gh pr merge 123 --auto --squash
```

**Delete branch after merge:**

```bash
gh pr merge 123 --squash --delete-branch
```

**Admin merge (bypass protections):**

```bash
gh pr merge 123 --squash --admin
# ⚠️ Use with extreme caution!
```

### Step 4: Verify Merge

After merging, confirm success:

```bash
# Check PR status
gh pr view 123 --json state,merged,mergedAt

# Verify base branch updated
git fetch origin
git log origin/main -1 --oneline

# Check if branch deleted
gh api repos/:owner/:repo/branches/feature-name 2>&1 | grep -q "Not Found"
```

### Step 5: Report Success

Present merge result:

```markdown
✓ PR #123 merged successfully

Strategy: Squash and merge
Merged by: @username
Merged at: 2024-01-01 15:30:00
Branch deleted: Yes

Commit: abc1234 - "Add user authentication"
Base branch: main updated

🔗 [View merged PR](https://github.com/owner/repo/pull/123)
```

## Common Scenarios

### Scenario 1: Simple Squash Merge

```bash
# Verify PR is ready
gh pr view 123 --json reviewDecision,mergeable
# reviewDecision: "APPROVED", mergeable: "MERGEABLE"

# Check CI
gh pr checks 123
# All checks passed ✓

# Merge with squash
gh pr merge 123 --squash --delete-branch

echo "✓ PR #123 merged and branch deleted"
```

### Scenario 2: Auto-Merge When Checks Pass

```bash
# Enable auto-merge for approved PR waiting on CI
gh pr merge 123 --auto --squash

echo "⏳ Auto-merge enabled. PR will merge when all checks pass."

# Watch check progress
gh pr checks 123 --watch
```

### Scenario 3: Merge with Custom Message

```bash
# Merge with detailed commit message
gh pr merge 123 --squash \
  --subject "feat: add user authentication system" \
  --body "$(cat <<'EOF'
Implements comprehensive authentication:

Features:
- JWT token-based auth
- Secure password hashing
- Session management
- OAuth2 integration

Breaking Changes:
- Auth header format changed from Bearer to JWT

Migration Guide:
Update client auth headers to use new JWT format.

Closes #45, #67
Related: #23

Co-authored-by: Alice <alice@example.com>
Co-authored-by: Bob <bob@example.com>
EOF
)"
```

### Scenario 4: Merge with Rebase for Clean History

```bash
# For PRs with clean, meaningful commits
gh pr view 123 --json commits

# Verify each commit is good
gh pr view 123 --json commits \
  | jq -r '.commits[] | .messageHeadline'

# Merge with rebase to preserve commits
gh pr merge 123 --rebase --delete-branch
```

### Scenario 5: Conditional Merge Based on Checks

```bash
# Only merge if all conditions met
PR_NUM=123

# Get PR data
PR_DATA=$(gh pr view $PR_NUM --json \
  reviewDecision,mergeable,statusCheckRollup,isDraft)

REVIEW=$(echo "$PR_DATA" | jq -r '.reviewDecision')
MERGEABLE=$(echo "$PR_DATA" | jq -r '.mergeable')
IS_DRAFT=$(echo "$PR_DATA" | jq -r '.isDraft')
CHECKS=$(echo "$PR_DATA" | jq -r '.statusCheckRollup[] | select(.isRequired == true) | .conclusion')

# Validate
if [[ "$REVIEW" == "APPROVED" ]] && \
   [[ "$MERGEABLE" == "MERGEABLE" ]] && \
   [[ "$IS_DRAFT" == "false" ]] && \
   ! echo "$CHECKS" | grep -qv "SUCCESS"; then

  gh pr merge $PR_NUM --squash --delete-branch
  echo "✓ PR merged successfully"
else
  echo "⚠ PR not ready to merge:"
  echo "  Review: $REVIEW"
  echo "  Mergeable: $MERGEABLE"
  echo "  Draft: $IS_DRAFT"
  echo "  Checks: $CHECKS"
  exit 1
fi
```

### Scenario 6: Batch Merge Approved PRs

```bash
# Merge all approved PRs from a milestone
gh pr list --json number,reviewDecision,mergeable,milestone \
  | jq -r '.[] |
    select(.reviewDecision == "APPROVED") |
    select(.mergeable == "MERGEABLE") |
    select(.milestone.title == "v1.0") |
    .number' \
  | while read pr; do
    echo "Merging PR #$pr..."
    gh pr merge $pr --squash --delete-branch
    sleep 2  # Rate limiting
  done
```

## Advanced Options

### Merge Queue

For repositories with merge queues enabled:

```bash
# Add to merge queue
gh pr merge 123 --auto --squash

# Check queue position
gh api repos/:owner/:repo/pulls/123/merge_queue \
  | jq '.position'
```

### Protected Branch Bypass

For admin users only:

```bash
# Bypass branch protections (use with extreme caution)
gh pr merge 123 --admin --squash

# This bypasses:
# - Required reviews
# - Status checks
# - Restrictions
```

### Merge with Required Linear History

```bash
# For repos requiring linear history
gh pr merge 123 --rebase

# Or squash to maintain linearity
gh pr merge 123 --squash
```

### Merge Dependabot PRs

```bash
# Auto-merge approved Dependabot PRs
gh pr list --author app/dependabot \
  --json number,reviewDecision \
  | jq -r '.[] | select(.reviewDecision == "APPROVED") | .number' \
  | while read pr; do
    gh pr merge $pr --auto --squash --delete-branch
  done
```

## Error Handling

**Error: "Pull request is not mergeable"**

- Cause: Merge conflicts exist
- Solution: Resolve conflicts first

```bash
git fetch origin
git checkout feature-branch
git merge origin/main
# Resolve conflicts
git push
```

**Error: "Required status checks have not succeeded"**

- Cause: CI checks failing or pending
- Solution: Wait for checks or fix failures

```bash
gh pr checks 123 --watch
```

**Error: "Review required"**

- Cause: PR needs approval
- Solution: Request reviews

```bash
gh pr review 123 --approve
# Or request from others:
gh pr edit 123 --add-reviewer user1,user2
```

**Error: "Branch protection rule violations"**

- Cause: Branch protections not met
- Solution: Satisfy requirements or use admin override (if authorized)

**Error: "Pull request is closed"**

- Cause: PR already merged or closed
- Solution: Verify PR status

```bash
gh pr view 123 --json state,merged
```

**Error: "Auto-merge not allowed"**

- Cause: Repository doesn't have auto-merge enabled
- Solution: Use standard merge or enable in repository settings

## Best Practices

1. **Always validate first**: Check reviews, checks, and mergeable status
2. **Choose appropriate strategy**:
   - Squash for feature branches with messy history
   - Rebase for clean commits
   - Merge for preserving exact history
3. **Delete branches**: Use `--delete-branch` to clean up
4. **Write good merge messages**: Provide context for squash merges
5. **Use auto-merge for trusted PRs**: Let CI complete before merging
6. **Never force merge**: Avoid `--admin` unless absolutely necessary
7. **Verify after merge**: Check base branch was updated
8. **Follow team conventions**: Use team's preferred merge strategy
9. **Wait for required checks**: Don't bypass failing checks
10. **Review before merge**: One final check of the code

## Merge Strategy Decision Tree

```
Is commit history clean and meaningful?
├─ Yes → Use --rebase
└─ No
   ├─ Multiple WIP commits? → Use --squash
   └─ Want to preserve exact history? → Use --merge

Does repository require linear history?
├─ Yes → Use --squash or --rebase
└─ No → Any strategy works

Is this a hotfix or emergency?
├─ Yes → Use --merge (preserve all context)
└─ No → Follow normal team conventions
```

## Related Skills

- `gh-pr-view` - View PR details before merging
- `gh-pr-create` - Create pull requests
- `gh-pr-review` - Review and approve PRs
- `gh-pr-ready` - Mark draft as ready

## Limitations

- Requires write access to repository
- Cannot merge if branch protections not satisfied (without admin override)
- Auto-merge requires repository setting enabled
- Squash merge may lose commit-level context
- Rebase may cause issues with force-pushed branches

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_pr_merge
- Merge strategies: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges
- Branch protections: https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
