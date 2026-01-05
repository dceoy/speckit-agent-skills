---
name: gh-pr-view
description: View GitHub pull request details using gh CLI. Shows PR status, reviews, checks, comments, and metadata. Use when user wants to inspect PR information.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: Create PR
    agent: gh-pr-create
    prompt: Create a new pull request
    send: true
  - label: Mark Ready
    agent: gh-pr-ready
    prompt: Mark this PR as ready for review
    send: true
---

# GitHub PR View Skill

View comprehensive pull request information using the `gh` CLI.

## When to Use

- User asks "show me the PR" or "what's the status of PR #123"
- User wants to see PR reviews, checks, or comments
- Before merging, to verify PR is approved and ready
- To check CI/CD status on a PR
- To see PR metadata (assignees, labels, reviewers)

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
gh pr view 123
```

**If on a branch with PR:**

```bash
gh pr view
```

**If user provides PR URL:**

```bash
gh pr view https://github.com/owner/repo/pull/123
```

### Step 2: Fetch PR Details

Get comprehensive JSON data:

```bash
gh pr view 123 --json \
  number,title,body,state,isDraft,\
  author,assignees,reviewers,\
  labels,milestone,\
  createdAt,updatedAt,mergedAt,\
  baseRefName,headRefName,\
  mergeable,merged,\
  additions,deletions,\
  url,commits,reviews,\
  statusCheckRollup
```

### Step 3: Format and Present

Parse the JSON and present in human-readable format:

```markdown
## PR #123: Title Here

**Status**: Open | Draft | Merged | Closed
**Author**: @username
**Created**: 2 days ago
**Updated**: 1 hour ago

**Base**: main ← **Head**: feature-branch
**Changes**: +150 -30 (180 lines changed)
**Mergeable**: Yes | No | Conflicts

### Reviews

- ✓ @reviewer1 approved (2 hours ago)
- 🔄 @reviewer2 requested changes (1 day ago)
- ⏳ @reviewer3 review requested

### Checks

- ✓ CI/CD (passed)
- ✓ Tests (passed)
- ⚠ Coverage (78% - below threshold)
- ❌ Linting (failed)

### Labels

`bug` `priority-high` `ready-for-review`

### Assignees

@user1, @user2

🔗 [View on GitHub](https://github.com/owner/repo/pull/123)
```

### Step 4: Show Additional Details (if needed)

**View diff:**

```bash
gh pr diff 123
```

**View commits:**

```bash
gh pr view 123 --json commits --jq '.commits[] | "\(.oid[0:7]) \(.messageHeadline)"'
```

**View comments:**

```bash
gh pr view 123 --comments
```

**View CI checks:**

```bash
gh pr checks 123
```

## Common Scenarios

### Scenario 1: Quick Status Check

```bash
# Show basic PR info
gh pr view 123

# Output:
# Feature: Add user authentication #123
# Open • user wants to merge 3 commits into main from feature-auth
#
# Summary of changes...
#
# View this pull request on GitHub: https://github.com/...
```

### Scenario 2: Detailed Review Status

```bash
# Get review status
gh pr view 123 --json reviews,reviewDecision \
  | jq -r '.reviews[] | "\(.author.login): \(.state)"'

# Get review decision
gh pr view 123 --json reviewDecision --jq '.reviewDecision'
# Output: APPROVED | CHANGES_REQUESTED | REVIEW_REQUIRED
```

### Scenario 3: Check CI/CD Status

```bash
# Watch checks in real-time
gh pr checks 123 --watch

# Get check status summary
gh pr checks 123 --json \
  | jq -r '.[] | "\(.name): \(.conclusion)"'
```

### Scenario 4: View PR Timeline

```bash
# Get complete timeline
gh pr view 123 --json \
  createdAt,updatedAt,mergedAt,closedAt,\
  timelineItems --jq '{
    created: .createdAt,
    updated: .updatedAt,
    merged: .mergedAt,
    timeline: .timelineItems
  }'
```

### Scenario 5: Compare with Base

```bash
# View diff stats
gh pr diff 123 --stat

# View full diff
gh pr diff 123

# View specific file
gh pr diff 123 --patch | grep -A 20 "src/auth.ts"
```

### Scenario 6: List All PRs

```bash
# List open PRs
gh pr list --state open

# List my PRs
gh pr list --author "@me"

# List draft PRs
gh pr list --json number,title,isDraft \
  | jq '.[] | select(.isDraft == true)'

# List PRs by label
gh pr list --label "priority-high"

# List PRs needing review
gh pr list --search "is:open review:required"
```

### Scenario 7: View PR Comments and Discussion

```bash
# Show all comments
gh pr view 123 --comments

# Show only review comments
gh pr view 123 --json reviews \
  | jq -r '.reviews[] | "\(.author.login) (\(.state)):\n\(.body)\n"'
```

## Advanced Queries

### Check if PR is Mergeable

```bash
# Check mergeable status
MERGEABLE=$(gh pr view 123 --json mergeable --jq '.mergeable')

if [ "$MERGEABLE" == "MERGEABLE" ]; then
  echo "✓ PR can be merged"
elif [ "$MERGEABLE" == "CONFLICTING" ]; then
  echo "⚠ PR has merge conflicts"
else
  echo "? Merge status unknown"
fi
```

### Get PR Metrics

```bash
# Calculate review turnaround time
gh pr view 123 --json createdAt,reviews \
  | jq -r '
    .reviews[0].submittedAt as $first_review |
    .createdAt as $created |
    (($first_review | fromdateiso8601) - ($created | fromdateiso8601)) / 3600 |
    floor | "\(.) hours to first review"
  '

# Count review cycles
gh pr view 123 --json reviews \
  | jq '.reviews | group_by(.author.login) | length | "Reviews from \(.) different people"'
```

### Filter by Review State

```bash
# Find PRs approved by specific user
gh pr list --json number,title,reviews \
  | jq '.[] | select(.reviews[] | select(.author.login == "user1" and .state == "APPROVED"))'

# Find PRs with changes requested
gh pr list --search "is:open review:changes-requested"
```

### Check Required Status Checks

```bash
# Get required checks status
gh pr view 123 --json statusCheckRollup \
  | jq -r '.statusCheckRollup[] |
    select(.isRequired == true) |
    "\(.name): \(.conclusion // .status)"'
```

## Formatting Options

### JSON Output

```bash
# Raw JSON for scripting
gh pr view 123 --json number,title,state,reviews

# Pretty JSON
gh pr view 123 --json number,title,state | jq '.'

# Specific fields
gh pr view 123 --json title --jq '.title'
```

### Template Output

```bash
# Custom format using Go templates
gh pr view 123 --template '
PR #{{.number}}: {{.title}}
Status: {{.state}}
Author: {{.author.login}}
Reviews: {{range .reviews}}{{.author.login}}({{.state}}) {{end}}
'
```

### Web View

```bash
# Open PR in browser
gh pr view 123 --web

# Open PR comments in browser
gh pr view 123 --web --comments
```

## Tips

- **Use JSON for scripting**: Parse with `jq` for automation
- **Watch checks**: Use `--watch` flag to monitor CI in real-time
- **Web view for details**: Open complex PRs in browser with `--web`
- **Cache results**: Store `gh pr view` output for repeated queries
- **Filter with jq**: Powerful JSON filtering for specific data
- **Combine with git**: Use `gh pr view` + `git show` for complete picture

## Error Handling

**Error: "Pull request not found"**

- Cause: PR doesn't exist or you don't have access
- Solution: Verify PR number with `gh pr list`

**Error: "No pull requests found"**

- Cause: Current branch doesn't have a PR
- Solution: Specify PR number or create one with `gh pr create`

**Error: "GraphQL error"**

- Cause: API rate limit or network issue
- Solution: Check `gh auth status` and wait/retry

**Error: "Not authorized"**

- Cause: Not authenticated or insufficient permissions
- Solution: Run `gh auth login` or check repository access

## Best Practices

1. **Check before merge**: Always view PR details before merging
2. **Verify reviews**: Ensure required reviewers have approved
3. **Check CI status**: Don't merge with failing checks
4. **Read comments**: Review all discussion threads
5. **Monitor conflicts**: Address merge conflicts promptly
6. **Track changes**: Note additions/deletions for impact assessment
7. **Use JSON for automation**: Script repetitive PR checks
8. **Web view for complex reviews**: Use browser for detailed code review
9. **Check required checks**: Verify all required status checks pass
10. **Review timeline**: Understand PR history before action

## Output Examples

### Basic View

```
Feature: Add user authentication #123
Open • alice wants to merge 5 commits into main from feature-auth
Draft

Summary:
Implements JWT-based authentication system.

Labels: feature, security
Assignees: alice, bob
Reviewers: charlie (APPROVED), diana (REVIEW_REQUESTED)

View this pull request on GitHub: https://github.com/org/repo/pull/123
```

### Checks View

```
All checks have passed
✓ CI/CD — 2m 34s
✓ Tests — 1m 12s
✓ Lint — 45s
✓ Security Scan — 3m 01s
```

### Diff Stats View

```
Showing diff for #123

src/auth/jwt.ts         | 145 +++++++++++++++++++++++++++++++++++
src/auth/middleware.ts  |  67 +++++++++++++++++
tests/auth.test.ts      | 234 +++++++++++++++++++++++++++++++++++++++++++++++++++
3 files changed, 446 insertions(+)
```

## Related Skills

- `gh-pr-create` - Create pull requests
- `gh-pr-ready` - Mark draft PR as ready
- `gh-pr-review` - Review and approve PRs
- `gh-pr-merge` - Merge approved PRs
- `gh-pr-edit` - Edit PR details

## Limitations

- Requires GitHub CLI installed and authenticated
- Some data requires repository access
- Large diffs may be truncated in terminal
- Real-time updates require polling or `--watch` flag
- Webhook/notification data not available via CLI

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_pr_view
- GitHub CLI formatting: https://cli.github.com/manual/gh_help_formatting
- jq manual: https://stedolan.github.io/jq/manual/
