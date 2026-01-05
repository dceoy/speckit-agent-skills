---
name: gh-issue-view
description: View GitHub issue details using gh CLI. Shows issue status, comments, timeline, and metadata. Use when user wants to inspect issue information or check status.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: Comment on Issue
    agent: gh-issue-comment
    prompt: Add a comment to this issue
    send: true
  - label: Close Issue
    agent: gh-issue-close
    prompt: Close this resolved issue
    send: true
  - label: Edit Issue
    agent: gh-issue-edit
    prompt: Update issue metadata
    send: true
---

# GitHub Issue View Skill

View comprehensive GitHub issue information using the `gh` CLI.

## When to Use

- User asks "show me issue #123" or "what's the status of this issue"
- User wants to see issue comments and discussion
- Before working on an issue, to understand requirements
- To check if issue is still open or already resolved
- To see who's assigned or what labels are applied

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

## Execution Workflow

### Step 1: Identify the Issue

**If user specifies issue number:**

```bash
gh issue view 123
```

**If user provides issue URL:**

```bash
gh issue view https://github.com/owner/repo/issues/123
```

**If searching for issues:**

```bash
gh issue list --search "login bug"
```

### Step 2: Fetch Issue Details

Get comprehensive JSON data:

```bash
gh issue view 123 --json \
  number,title,body,state,\
  author,assignees,labels,\
  milestone,createdAt,updatedAt,closedAt,\
  comments,url
```

### Step 3: Format and Present

Parse and present in human-readable format:

```markdown
## Issue #123: Fix login button on Safari

**Status**: Open | Closed
**Author**: @username
**Created**: 3 days ago
**Updated**: 2 hours ago
**Closed**: N/A (or timestamp)

**Assignees**: @alice, @bob
**Labels**: `bug` `priority-high` `mobile`
**Milestone**: v2.1

### Description

Users cannot log in when using Safari browser on iOS.
Login button doesn't respond when clicked.

### Comments (3)

**@alice** (2 days ago):
I can reproduce this on iOS 17. Investigating...

**@bob** (1 day ago):
Found the issue - event listener not attached correctly.
Working on a fix.

**@charlie** (2 hours ago):
Fixed in PR #234. Testing now.

🔗 [View on GitHub](https://github.com/owner/repo/issues/123)
```

### Step 4: Show Additional Details (if needed)

**View with comments:**

```bash
gh issue view 123 --comments
```

**View timeline:**

```bash
gh issue view 123 --json timelineItems \
  | jq -r '.timelineItems[] | "\(.createdAt) - \(.typename): \(.body)"'
```

**Check linked PRs:**

```bash
gh issue view 123 --json timelineItems \
  | jq -r '.timelineItems[] | select(.typename == "CrossReferencedEvent") | .source.url'
```

## Common Scenarios

### Scenario 1: Quick Issue Check

```bash
# Simple view
gh issue view 123

# Output:
# Fix login button on Safari #123
# Open • alice opened 3 days ago • 5 comments
#
# Users cannot log in when using Safari browser on iOS...
#
# View this issue on GitHub: https://github.com/...
```

### Scenario 2: Detailed Issue Analysis

```bash
# Get full details with comments
gh issue view 123 --comments

# Check who's working on it
gh issue view 123 --json assignees \
  | jq -r '.assignees[] | .login'

# Check labels and priority
gh issue view 123 --json labels \
  | jq -r '.labels[] | .name'

# See when it was created
gh issue view 123 --json createdAt,updatedAt \
  | jq -r '"Created: \(.createdAt)\nUpdated: \(.updatedAt)"'
```

### Scenario 3: List All Open Issues

```bash
# List all open issues
gh issue list --state open

# List my issues
gh issue list --assignee "@me"

# List high priority bugs
gh issue list --label "bug,priority-high"

# List issues in milestone
gh issue list --milestone "v2.0"

# Custom columns
gh issue list --json number,title,assignees,labels \
  --template '{{range .}}{{.number}}: {{.title}} [@{{range .assignees}}{{.login}} {{end}}]{{"\n"}}{{end}}'
```

### Scenario 4: Search for Issues

```bash
# Search by keyword
gh issue list --search "authentication"

# Search with filters
gh issue list --search "login is:open label:bug"

# Search by author
gh issue list --author "alice"

# Search recently updated
gh issue list --search "updated:>2024-01-01"
```

### Scenario 5: Check Issue Status and Blockers

```bash
# Get issue with dependencies
ISSUE_BODY=$(gh issue view 123 --json body --jq '.body')

# Parse blocking issues
echo "$ISSUE_BODY" | grep -E "Blocks #[0-9]+" -o | grep -o "[0-9]*"

# Parse dependencies
echo "$ISSUE_BODY" | grep -E "Depends on #[0-9]+" -o | grep -o "[0-9]*"

# Check if blockers are resolved
for blocker in $(echo "$ISSUE_BODY" | grep -E "Depends on #[0-9]+" -o | grep -o "[0-9]*"); do
  STATUS=$(gh issue view $blocker --json state --jq '.state')
  echo "Issue #$blocker: $STATUS"
done
```

### Scenario 6: View Issue Timeline

```bash
# Get full timeline
gh issue view 123 --json timelineItems \
  | jq -r '.timelineItems[] | "\(.createdAt | split("T")[0]) - \(.actor.login // "system"): \(.typename)"'

# Example output:
# 2024-01-01 - alice: IssueComment
# 2024-01-02 - bob: LabeledEvent
# 2024-01-03 - system: CrossReferencedEvent
# 2024-01-04 - alice: AssignedEvent
```

### Scenario 7: Export Issue to File

```bash
# Export issue details to markdown
gh issue view 123 > issue-123.md

# Export with comments
gh issue view 123 --comments > issue-123-full.md

# Export JSON for processing
gh issue view 123 --json \
  number,title,body,state,author,assignees,labels,comments,createdAt \
  > issue-123.json
```

## Advanced Queries

### Check Issue Age and Activity

```bash
# Calculate issue age
CREATED=$(gh issue view 123 --json createdAt --jq '.createdAt')
NOW=$(date -u +%s)
CREATED_TS=$(date -d "$CREATED" +%s 2>/dev/null || date -j -f "%Y-%m-%dT%H:%M:%SZ" "$CREATED" +%s)
AGE_DAYS=$(( ($NOW - $CREATED_TS) / 86400 ))

echo "Issue is $AGE_DAYS days old"

# Check if stale (no activity in 30 days)
UPDATED=$(gh issue view 123 --json updatedAt --jq '.updatedAt')
UPDATED_TS=$(date -d "$UPDATED" +%s 2>/dev/null || date -j -f "%Y-%m-%dT%H:%M:%SZ" "$UPDATED" +%s)
STALE_DAYS=$(( ($NOW - $UPDATED_TS) / 86400 ))

if [ $STALE_DAYS -gt 30 ]; then
  echo "⚠️ Issue is stale (no activity for $STALE_DAYS days)"
fi
```

### Find Related Issues

```bash
# Find issues with same labels
LABELS=$(gh issue view 123 --json labels --jq '.labels[].name' | paste -sd,)

gh issue list --label "$LABELS" --json number,title \
  | jq -r '.[] | select(.number != 123) | "#\(.number): \(.title)"'
```

### Check Resolution Status

```bash
# Check if issue is closed and how
gh issue view 123 --json state,stateReason \
  | jq -r 'if .state == "CLOSED" then "Closed as: \(.stateReason)" else "Still open" end'

# stateReason can be: COMPLETED, NOT_PLANNED, REOPENED
```

### Analyze Comment Activity

```bash
# Count comments
COMMENT_COUNT=$(gh issue view 123 --json comments --jq '.comments | length')
echo "Total comments: $COMMENT_COUNT"

# List participants
gh issue view 123 --json comments \
  | jq -r '.comments[].author.login' | sort -u

# Find last comment timestamp
gh issue view 123 --json comments \
  | jq -r '.comments[-1].createdAt // "No comments"'
```

### Issue Metrics Dashboard

```bash
# Create issue metrics dashboard
echo "=== Issue Metrics ==="

# Total issues
TOTAL=$(gh issue list --limit 1000 | wc -l)
echo "Total open issues: $TOTAL"

# By label
echo -e "\nBy Label:"
gh issue list --json labels --jq '.[].labels[].name' | sort | uniq -c | sort -rn | head -5

# By assignee
echo -e "\nBy Assignee:"
gh issue list --json assignees --jq '.[].assignees[].login' | sort | uniq -c | sort -rn | head -5

# By milestone
echo -e "\nBy Milestone:"
gh issue list --json milestone --jq '.[].milestone.title' | grep -v null | sort | uniq -c | sort -rn
```

## Formatting Options

### JSON Output

```bash
# Raw JSON
gh issue view 123 --json number,title,state,assignees

# Pretty JSON
gh issue view 123 --json number,title,state | jq '.'

# Specific field
gh issue view 123 --json title --jq '.title'
```

### Custom Template

```bash
# Go template formatting
gh issue view 123 --template '
Issue #{{.number}}: {{.title}}
State: {{.state}}
Author: {{.author.login}}
Assignees: {{range .assignees}}@{{.login}} {{end}}
Labels: {{range .labels}}{{.name}} {{end}}
'
```

### Web View

```bash
# Open in browser
gh issue view 123 --web

# Open comments in browser
gh issue view 123 --web --comments
```

## Filtering and Sorting

### List with Filters

```bash
# Open bugs only
gh issue list --label "bug" --state open

# My assigned issues
gh issue list --assignee "@me"

# Issues without assignee
gh issue list --search "no:assignee is:open"

# Issues in current milestone
MILESTONE=$(gh api repos/:owner/:repo/milestones --jq '.[0].title')
gh issue list --milestone "$MILESTONE"

# Recently updated
gh issue list --search "updated:>2024-01-01 is:open"

# Sort by created date
gh issue list --json number,title,createdAt \
  | jq -r 'sort_by(.createdAt) | reverse | .[] | "#\(.number): \(.title)"'
```

### Complex Searches

```bash
# Find old open issues
gh issue list --search "is:open created:<2023-01-01"

# Find issues with no comments
gh issue list --search "is:open comments:0"

# Find issues with many comments (active discussion)
gh issue list --search "is:open comments:>10"

# Find issues by keyword in body
gh issue list --search "authentication in:body"
```

## Tips

- **Use web view for detailed review**: `--web` opens in browser
- **Cache results for dashboards**: Store `gh issue list` output
- **Filter by multiple criteria**: Combine labels, milestones, assignees
- **Use JSON for automation**: Parse with `jq` for scripts
- **Check dependencies**: Look for "Depends on #" in issue body
- **Monitor timeline**: Track issue history with timeline
- **Export for reporting**: Save issues to files for records

## Error Handling

**Error: "Issue not found"**

- Cause: Issue doesn't exist or you don't have access
- Solution: Verify issue number with `gh issue list`

**Error: "Not authorized"**

- Cause: Not authenticated or no repository access
- Solution: Run `gh auth login` or request access

**Error: "GraphQL error"**

- Cause: API rate limit or network issue
- Solution: Check `gh auth status` and wait/retry

## Best Practices

1. **Check before working**: Always view issue before starting work
2. **Read all comments**: Understand full context and discussion
3. **Note dependencies**: Check for blocking or related issues
4. **Review labels**: Understand priority and categorization
5. **Check assignees**: Coordinate if already assigned
6. **Look for PRs**: See if work is already in progress
7. **Note milestones**: Understand release timeline
8. **Export for records**: Save issue details before closing
9. **Use web view for complex issues**: Browser better for long discussions
10. **Monitor stale issues**: Check update timestamps

## Output Examples

### Basic View

```
Fix login button on Safari #123
Open • alice opened 3 days ago • 5 comments

Users cannot log in when using Safari browser on iOS.
Login button doesn't respond when clicked.

Labels: bug, priority-high, mobile
Assignees: alice, bob
Milestone: v2.1

View this issue on GitHub: https://github.com/org/repo/issues/123
```

### With Comments

```
Fix login button on Safari #123
Open • alice opened 3 days ago • 5 comments

[Issue body...]

---

alice commented 2 days ago
I can reproduce this on iOS 17. Investigating the event handlers.

bob commented 1 day ago
Found the root cause - addEventListener not working in strict mode.
Working on a fix in PR #234.

---

View this issue on GitHub: https://github.com/org/repo/issues/123
```

### List View

```
Showing 5 of 23 open issues in owner/repo

#123  Fix login button on Safari                  bug, priority-high
#122  Add export to CSV feature                   enhancement
#121  Update dependencies                         maintenance
#120  Memory leak in auth module                  bug, performance
#119  Documentation for API endpoints             docs
```

## Related Skills

- `gh-issue-create` - Create new issues
- `gh-issue-comment` - Add comments to issues
- `gh-issue-edit` - Update issue metadata
- `gh-issue-close` - Close/resolve issues
- `gh-issue-develop` - Start working on issue

## Limitations

- Large result sets may be paginated (use `--limit` to control)
- Comments may be truncated in terminal output
- Timeline data requires JSON output to access
- Reactions and detailed events not available in simple view
- No real-time updates (must re-run command)

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_view
- GitHub CLI formatting: https://cli.github.com/manual/gh_help_formatting
- GitHub search syntax: https://docs.github.com/en/search-github/searching-on-github/searching-issues-and-pull-requests
