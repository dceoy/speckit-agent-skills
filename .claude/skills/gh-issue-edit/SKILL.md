---
name: gh-issue-edit
description: Edit GitHub issue metadata using gh CLI. Update title, body, labels, assignees, milestones, and projects. Use when issue details need to be changed.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: View Issue
    agent: gh-issue-view
    prompt: View this updated issue
    send: true
  - label: Comment on Changes
    agent: gh-issue-comment
    prompt: Add comment explaining the changes
    send: true
---

# GitHub Issue Edit Skill

Edit GitHub issue metadata and content using the `gh` CLI.

## When to Use

- User says "edit issue #123" or "update the issue"
- Need to change issue title or description
- Add/remove labels, assignees, or milestone
- Fix typos or clarify requirements
- Update priority or categorization
- Reassign work to different team member

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

Verify repository access:

```bash
gh repo view --json viewerPermission --jq '.viewerPermission'
# Need write/triage access
```

## Execution Workflow

### Step 1: View Current State

Check current issue metadata:

```bash
gh issue view 123 --json \
  number,title,body,labels,assignees,milestone,state
```

### Step 2: Determine Changes Needed

Identify what needs to be updated:

- **Title**: Fix typos, clarify description
- **Body**: Add details, update requirements
- **Labels**: Add/remove categorization labels
- **Assignees**: Change who's working on it
- **Milestone**: Update release target
- **Project**: Move to different project board
- **State**: Reopen closed issue

### Step 3: Make Updates

**Edit title:**

```bash
gh issue edit 123 --title "Fix: Login button not responding on Safari"
```

**Edit body:**

```bash
gh issue edit 123 --body "$(cat <<'EOF'
## Updated Description

Users cannot log in when using Safari browser on iOS 17.

## Updated Requirements
- Must work on Safari 16+
- Should show error message if browser unsupported
- Need fallback for older iOS versions

## Additional Context
This is now blocking the v2.1 release.
EOF
)"
```

**Edit from file:**

```bash
gh issue edit 123 --body-file updated-description.md
```

**Add labels:**

```bash
gh issue edit 123 --add-label "priority-high,security"
```

**Remove labels:**

```bash
gh issue edit 123 --remove-label "needs-triage"
```

**Replace all labels:**

```bash
gh issue edit 123 --label "bug,priority-critical,security"
```

**Add assignees:**

```bash
gh issue edit 123 --add-assignee alice,bob
```

**Remove assignees:**

```bash
gh issue edit 123 --remove-assignee charlie
```

**Set milestone:**

```bash
gh issue edit 123 --milestone "v2.1"
```

**Remove milestone:**

```bash
gh issue edit 123 --milestone ""
```

**Add to project:**

```bash
gh issue edit 123 --add-project "Q1 Roadmap"
```

**Remove from project:**

```bash
gh issue edit 123 --remove-project "Backlog"
```

### Step 4: Verify Changes

Confirm updates were applied:

```bash
gh issue view 123 --json title,labels,assignees,milestone \
  | jq '.'
```

### Step 5: Document Changes (Optional)

Add comment explaining the changes:

```bash
gh issue comment 123 --body "$(cat <<'EOF'
Updated issue metadata:
- Changed priority to critical
- Added security label
- Assigned to @alice for immediate attention
- Moved to v2.1 milestone

Reason: This is blocking production deployment.
EOF
)"
```

### Step 6: Report to User

```markdown
✓ Issue #123 updated successfully

Changes:

- Title: "Login bug" → "Fix: Login button not responding on Safari"
- Labels: +priority-high, +security, -needs-triage
- Assignees: +alice, +bob
- Milestone: → v2.1

🔗 [View Issue](https://github.com/owner/repo/issues/123)
```

## Common Scenarios

### Scenario 1: Update Priority

```bash
# Increase priority
gh issue edit 123 \
  --remove-label "priority-low" \
  --add-label "priority-critical" \
  --add-assignee "team-lead"

gh issue comment 123 \
  --body "Escalating to critical - impacting production users."
```

### Scenario 2: Reassign Work

```bash
# Reassign from alice to bob
gh issue edit 123 \
  --remove-assignee alice \
  --add-assignee bob

gh issue comment 123 \
  --body "@bob Taking over from @alice. Context: [brief summary]"
```

### Scenario 3: Move to Different Milestone

```bash
# Move from v2.0 to v2.1
gh issue edit 123 --milestone "v2.1"

gh issue comment 123 \
  --body "Moving to v2.1 - won't make it into v2.0 release."
```

### Scenario 4: Clarify Requirements

```bash
# Update description with clearer requirements
gh issue edit 123 --body "$(cat <<'EOF'
## Problem
Users cannot export data when dataset >10,000 rows.

## Requirements
- Support exports up to 100,000 rows
- Show progress indicator for large exports
- Implement pagination for datasets >100k
- Add export format options (CSV, JSON, Excel)

## Acceptance Criteria
- [ ] Exports complete successfully for 100k rows
- [ ] Progress bar shows during export
- [ ] User can select export format
- [ ] Error message shown for datasets >100k

## Technical Notes
Consider streaming export to avoid memory issues.
EOF
)"

gh issue comment 123 \
  --body "Updated requirements based on product team feedback."
```

### Scenario 5: Fix Categorization

```bash
# Fix incorrect labels
gh issue edit 123 \
  --remove-label "enhancement" \
  --add-label "bug,regression"

gh issue comment 123 \
  --body "Correcting labels - this is a regression, not an enhancement."
```

### Scenario 6: Bulk Update Issues

```bash
# Add label to all issues in milestone
gh issue list --milestone "v2.0" --json number \
  | jq -r '.[].number' \
  | while read issue; do
    gh issue edit $issue --add-label "release-v2.0"
    echo "Added label to issue #$issue"
    sleep 1
  done
```

### Scenario 7: Reopen Closed Issue

```bash
# Reopen issue that was closed prematurely
gh issue reopen 123

gh issue edit 123 --add-label "reopened"

gh issue comment 123 --body "$(cat <<'EOF'
Reopening - issue is still occurring in production.

New occurrences:
- Production server A: 10 errors in last hour
- Production server B: 5 errors in last hour

Original fix in PR #234 didn't address all cases.
EOF
)"
```

## Advanced Usage

### Batch Update with Condition

```bash
# Update all bugs without assignee
gh issue list --label "bug" --search "no:assignee" --json number \
  | jq -r '.[].number' \
  | while read issue; do
    gh issue edit $issue --add-assignee "triage-team"
    echo "Assigned triage-team to issue #$issue"
    sleep 1
  done
```

### Update Based on PR Status

```bash
# When PR is merged, update related issue
PR_NUM=234
ISSUE_NUM=$(gh pr view $PR_NUM --json body | jq -r '.body' | grep -oE "#[0-9]+" | head -1 | tr -d '#')

if [ -n "$ISSUE_NUM" ]; then
  gh issue edit $ISSUE_NUM \
    --add-label "fixed-pending-deployment" \
    --remove-label "in-progress"

  gh issue comment $ISSUE_NUM \
    --body "Fix merged in PR #$PR_NUM. Pending deployment to production."
fi
```

### Sync Labels Across Related Issues

```bash
# Apply same labels to related issues
LABELS=$(gh issue view 123 --json labels --jq '.labels[].name' | paste -sd,)

for issue in 124 125 126; do
  gh issue edit $issue --label "$LABELS"
  echo "Synced labels to issue #$issue"
done
```

### Update from Template

```bash
# Use template for consistent updates
cat > issue-template.md <<'EOF'
## Problem Statement
{problem}

## Acceptance Criteria
{criteria}

## Technical Notes
{notes}
EOF

# Fill and apply
sed -e "s/{problem}/Clear problem description/" \
    -e "s/{criteria}/Detailed criteria/" \
    -e "s/{notes}/Implementation notes/" \
    issue-template.md > filled-issue.md

gh issue edit 123 --body-file filled-issue.md
```

### Progressive Enhancement

```bash
# Incrementally add details as you learn more
gh issue edit 123 --body "$(cat <<EOF
$(gh issue view 123 --json body --jq '.body')

## Additional Findings
- Also affects Firefox
- Workaround: clear cookies
- Root cause: session storage issue
EOF
)"
```

## Edit Strategies

### Minimal Edits

Only change what's necessary. Preserve original context.

```bash
# Just fix typo in title
gh issue edit 123 --title "Fix login button (not 'button login')"
```

### Comprehensive Updates

Major revisions with full rewrite.

```bash
# Complete rewrite after investigation
gh issue edit 123 --body-file comprehensive-update.md
gh issue comment 123 --body "Updated issue with full investigation findings."
```

### Incremental Updates

Add information without removing existing content.

```bash
# Append new information
CURRENT_BODY=$(gh issue view 123 --json body --jq '.body')
gh issue edit 123 --body "$(cat <<EOF
$CURRENT_BODY

## Update $(date +%Y-%m-%d)
New information discovered during testing...
EOF
)"
```

## Tips

- **Explain changes**: Add comment when making significant edits
- **Preserve history**: Don't delete useful information
- **Be specific**: Change only what needs changing
- **Use labels wisely**: Don't over-label, keep it simple
- **Assign appropriately**: Only assign people who will actually work on it
- **Update milestones**: Move issues when priorities change
- **Fix typos promptly**: Clean up mistakes quickly
- **Keep title clear**: Title should summarize the issue

## Error Handling

**Error: "Issue not found"**

- Cause: Issue doesn't exist or no access
- Solution: Verify issue number with `gh issue list`

**Error: "Not authorized"**

- Cause: Insufficient permissions
- Solution: Request write/triage access

**Error: "Invalid label"**

- Cause: Label doesn't exist
- Solution: Check available labels with `gh label list`

**Error: "Invalid assignee"**

- Cause: User not a collaborator
- Solution: Verify collaborators with `gh api repos/:owner/:repo/collaborators`

**Error: "Invalid milestone"**

- Cause: Milestone doesn't exist
- Solution: List milestones with `gh api repos/:owner/:repo/milestones`

**Error: "Project not found"**

- Cause: Project doesn't exist or no access
- Solution: List projects with `gh project list`

## Best Practices

1. **Explain major changes**: Add comment when significantly updating
2. **Don't delete useful info**: Preserve investigation notes and context
3. **Use labels consistently**: Follow team's labeling conventions
4. **Keep titles concise**: 50-70 characters is ideal
5. **Update proactively**: Fix issues and typos as you find them
6. **Reassign carefully**: Only assign when someone commits to work
7. **Track milestone changes**: Document why issues are moved
8. **Be precise with labels**: Add only relevant, accurate labels
9. **Maintain formatting**: Use consistent markdown style
10. **Version important changes**: Note what changed and when

## Common Label Patterns

**Type Labels:**

- `bug`, `enhancement`, `feature`, `documentation`, `question`

**Priority Labels:**

- `priority-critical`, `priority-high`, `priority-medium`, `priority-low`

**Status Labels:**

- `in-progress`, `blocked`, `needs-review`, `ready-for-dev`

**Area Labels:**

- `frontend`, `backend`, `security`, `performance`, `ux`

## Related Skills

- `gh-issue-view` - View current issue state before editing
- `gh-issue-comment` - Document why changes were made
- `gh-issue-close` - Close issues after editing
- `gh-issue-create` - Create new issues with correct metadata

## Limitations

- Cannot edit other users' comments (only issue body)
- Cannot change issue number or repository
- Cannot transfer to different repository via edit (use `gh issue transfer`)
- Cannot batch edit all fields at once (requires multiple commands)
- Cannot edit locked issues without unlocking first

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_edit
- Issue management: https://docs.github.com/en/issues/tracking-your-work-with-issues
- Labels: https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/managing-labels
