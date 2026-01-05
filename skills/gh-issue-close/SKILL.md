---
name: gh-issue-close
description: Close and resolve GitHub issues using gh CLI. Mark issues as completed, won't fix, or duplicate. Use when issue is resolved, invalid, or no longer relevant.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: View Issue
    agent: gh-issue-view
    prompt: View this closed issue
    send: true
  - label: Reopen Issue
    agent: gh-issue-edit
    prompt: Reopen this issue
    send: true
---

# GitHub Issue Close Skill

Close and resolve GitHub issues using the `gh` CLI with proper resolution tracking.

## When to Use

- User says "close issue #123" or "resolve this issue"
- Work is complete and issue should be marked resolved
- Issue is invalid, duplicate, or won't be fixed
- Issue is no longer relevant or out of scope
- After merging PR that fixes the issue

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

Verify repository access (need write/triage permission):

```bash
gh repo view --json viewerPermission --jq '.viewerPermission'
```

## Execution Workflow

### Step 1: Verify Issue Status

Check if issue is open and should be closed:

```bash
# View current status
gh issue view 123 --json number,title,state,stateReason

# Ensure it's open
STATE=$(gh issue view 123 --json state --jq '.state')
if [ "$STATE" != "OPEN" ]; then
  echo "Issue #123 is already closed"
  exit 0
fi
```

### Step 2: Determine Close Reason

GitHub supports different close reasons:

**COMPLETED** (default):

- Issue was resolved/fixed
- Work is done
- PR merged that fixes it

**NOT_PLANNED**:

- Won't fix
- Out of scope
- Duplicate of another issue
- Invalid/spam

### Step 3: Add Resolution Comment

Before closing, add a comment explaining resolution:

```bash
gh issue comment 123 --body "$(cat <<'EOF'
Fixed in PR #234. Changes include:
- Updated authentication logic
- Added validation
- Fixed Safari compatibility

Deployed to production in v2.1.0.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

### Step 4: Close the Issue

**Close as completed:**

```bash
gh issue close 123
```

**Close as not planned:**

```bash
gh issue close 123 --reason "not planned"
```

**Close with comment:**

```bash
gh issue close 123 --comment "Fixed in PR #234. Deployed to production."
```

### Step 5: Verify Closure

Confirm issue was closed:

```bash
gh issue view 123 --json state,stateReason,closedAt \
  | jq -r '"State: \(.state)\nReason: \(.stateReason)\nClosed: \(.closedAt)"'
```

### Step 6: Report to User

Present the result:

```markdown
✓ Issue #123 closed successfully

Title: Fix login button on Safari
Reason: COMPLETED
Closed by: @username
Closed at: 2024-01-01 15:30:00

Resolution: Fixed in PR #234

🔗 [View Issue](https://github.com/owner/repo/issues/123)
```

## Common Scenarios

### Scenario 1: Close After PR Merge

```bash
# After PR #234 is merged
PR_NUM=234
ISSUE_NUM=$(gh pr view $PR_NUM --json body \
  | jq -r '.body' \
  | grep -oE "Closes #[0-9]+" \
  | grep -oE "[0-9]+")

if [ -n "$ISSUE_NUM" ]; then
  gh issue close $ISSUE_NUM \
    --comment "Fixed in PR #$PR_NUM and merged to main."
  echo "✓ Closed issue #$ISSUE_NUM"
fi
```

### Scenario 2: Close as Duplicate

```bash
# Issue is duplicate of #45
gh issue close 123 \
  --reason "not planned" \
  --comment "Duplicate of #45. Please follow that issue for updates."
```

### Scenario 3: Close as Won't Fix

```bash
gh issue close 123 \
  --reason "not planned" \
  --comment "$(cat <<'EOF'
After discussion, we've decided not to implement this feature because:
- Out of scope for current roadmap
- Conflicts with planned architecture changes
- Low user demand relative to effort required

Consider using [alternative solution] instead.
EOF
)"
```

### Scenario 4: Close as Invalid/Spam

```bash
gh issue close 123 \
  --reason "not planned" \
  --comment "Closing as invalid. Please reopen with more details if this is a legitimate issue."
```

### Scenario 5: Close Multiple Related Issues

```bash
# Close all issues fixed by PR #234
PR_BODY=$(gh pr view 234 --json body --jq '.body')

# Extract all "Closes #N" references
for issue in $(echo "$PR_BODY" | grep -oE "Closes #[0-9]+" | grep -oE "[0-9]+"); do
  echo "Closing issue #$issue..."
  gh issue close $issue \
    --comment "Fixed in PR #234. Merged to main and deployed."
  sleep 1  # Rate limiting
done
```

### Scenario 6: Close Stale Issues

```bash
# Close issues with no activity for 90+ days
gh issue list --search "updated:<$(date -d '90 days ago' +%Y-%m-%d)" \
  --json number,title,updatedAt \
  | jq -r '.[] | .number' \
  | while read issue; do
    echo "Closing stale issue #$issue..."
    gh issue close $issue \
      --reason "not planned" \
      --comment "Closing due to inactivity. Please reopen if still relevant."
    sleep 2
  done
```

### Scenario 7: Close with Verification Checklist

```bash
# Ensure all criteria met before closing
ISSUE_NUM=123

echo "Verifying issue can be closed..."

# Check if PR exists and is merged
PR_NUM=$(gh issue view $ISSUE_NUM --json body \
  | jq -r '.body' \
  | grep -oE "PR #[0-9]+" \
  | grep -oE "[0-9]+")

if [ -n "$PR_NUM" ]; then
  PR_STATE=$(gh pr view $PR_NUM --json state,merged --jq '"\(.state),\(.merged)"')
  if [[ "$PR_STATE" != "MERGED,true" ]]; then
    echo "⚠️ PR #$PR_NUM not merged yet"
    exit 1
  fi
fi

# Check if deployed
echo "Has this been deployed to production? (y/n)"
read deployed

if [ "$deployed" = "y" ]; then
  gh issue close $ISSUE_NUM \
    --comment "✅ Verified and deployed to production. All acceptance criteria met."
else
  echo "⏸ Not closing yet - waiting for deployment"
fi
```

## Advanced Options

### Close and Link to Documentation

```bash
gh issue close 123 \
  --comment "$(cat <<'EOF'
Resolved. Documentation updated:
- User guide: https://docs.example.com/auth
- API docs: https://api.example.com/auth
- Migration guide: https://docs.example.com/migration

See commit abc1234 for implementation details.
EOF
)"
```

### Close with Metrics

```bash
# Close with resolution time metrics
CREATED=$(gh issue view 123 --json createdAt --jq '.createdAt')
NOW=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

CREATED_TS=$(date -d "$CREATED" +%s 2>/dev/null || date -j -f "%Y-%m-%dT%H:%M:%SZ" "$CREATED" +%s)
NOW_TS=$(date -u +%s)
DAYS=$(( ($NOW_TS - $CREATED_TS) / 86400 ))

gh issue close 123 \
  --comment "Resolved in $DAYS days. Fixed in PR #234."
```

### Close with Release Note

```bash
gh issue close 123 \
  --comment "$(cat <<'EOF'
Fixed in version 2.1.0

**Release Notes:**
- Improved Safari compatibility
- Fixed authentication edge cases
- Enhanced error handling

Upgrade: `npm install app@2.1.0`
EOF
)"
```

### Bulk Close by Label

```bash
# Close all bugs marked as "fixed-pending-release"
gh issue list --label "fixed-pending-release" --json number \
  | jq -r '.[].number' \
  | while read issue; do
    gh issue close $issue \
      --comment "Fixed and deployed in latest release."
  done
```

### Close with Auto-notification

```bash
# Close and notify in Slack/Discord
ISSUE_URL=$(gh issue view 123 --json url --jq '.url')
ISSUE_TITLE=$(gh issue view 123 --json title --jq '.title')

gh issue close 123 --comment "Fixed in PR #234"

# Notify team (example webhook)
curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
  -H 'Content-Type: application/json' \
  -d "{\"text\":\"Issue resolved: $ISSUE_TITLE - $ISSUE_URL\"}"
```

## Close Reason Decision Tree

```
Is the work done?
├─ Yes → Close as COMPLETED
└─ No
   ├─ Is it a duplicate? → Close as NOT_PLANNED with "Duplicate of #N"
   ├─ Is it invalid/spam? → Close as NOT_PLANNED with "Invalid"
   ├─ Won't fix? → Close as NOT_PLANNED with explanation
   └─ Out of scope? → Close as NOT_PLANNED with reasoning
```

## Tips

- **Always add comment**: Explain why issue is being closed
- **Link to PR**: Reference PR that fixed the issue
- **Use correct reason**: COMPLETED vs NOT_PLANNED
- **Close after deployment**: Don't close until verified in production
- **Update documentation**: Link to updated docs in closing comment
- **Notify stakeholders**: Tag users who should know about resolution
- **Add release notes**: Note which version includes the fix
- **Clean up labels**: Remove "in-progress" labels before closing

## Error Handling

**Error: "Issue not found"**

- Cause: Issue doesn't exist or no access
- Solution: Verify issue number with `gh issue list`

**Error: "Not authorized"**

- Cause: Insufficient permissions to close issues
- Solution: Request write/triage access to repository

**Error: "Issue is already closed"**

- Cause: Issue was already closed
- Solution: Use `gh issue reopen 123` if needed

**Error: "GraphQL error"**

- Cause: API rate limit or network issue
- Solution: Wait and retry

## Best Practices

1. **Verify before closing**: Ensure work is actually complete
2. **Add resolution comment**: Always explain why closing
3. **Link to evidence**: Reference PRs, commits, or deployment
4. **Use correct reason**:
   - Use COMPLETED for resolved issues
   - Use NOT_PLANNED for won't fix, duplicates, invalid
5. **Close after deployment**: Don't close until verified in production
6. **Update related issues**: Close all related/duplicate issues
7. **Remove active labels**: Clear "in-progress", "help-wanted", etc.
8. **Tag participants**: Notify issue author and assignees
9. **Document lessons**: Add notes for future reference
10. **Track metrics**: Note resolution time for process improvement

## Reopen If Needed

If issue needs to be reopened:

```bash
gh issue reopen 123 --comment "Reopening - issue still occurring in production"
```

## Related Skills

- `gh-issue-view` - View issue before closing
- `gh-issue-comment` - Add resolution details
- `gh-issue-edit` - Update metadata before closing
- `gh-pr-view` - Verify PR that fixes issue

## Limitations

- Requires write/triage access to repository
- Cannot close issues in archived repositories
- Cannot change close reason after closing (must reopen and close again)
- Cannot close locked issues without unlock first

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_close
- Issue lifecycle: https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues
- State reasons: https://docs.github.com/en/issues/tracking-your-work-with-issues/closing-an-issue
