---
name: gh-issue-create
description: Create GitHub issues using gh CLI with templates, labels, and assignments. Use when user wants to report bugs, request features, or create tasks.
allowed-tools: Bash, Read, Grep, Glob
handoffs:
  - label: View Issue
    agent: gh-issue-view
    prompt: View the created issue
    send: true
  - label: Start Working on Issue
    agent: gh-issue-develop
    prompt: Create a branch and start working on this issue
    send: true
---

# GitHub Issue Create Skill

Create GitHub issues using the `gh` CLI with intelligent templates and automation.

## When to Use

- User says "create an issue" or "file a bug report"
- User wants to track a feature request
- User needs to document a task or problem
- User wants to create issues from errors or logs
- After discovering bugs during development

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

Verify repository access:

```bash
gh repo view --json viewerPermission --jq '.viewerPermission'
# Should be: WRITE, MAINTAIN, or ADMIN (or TRIAGE for issues)
```

## Execution Workflow

### Step 1: Determine Issue Type

Identify what kind of issue to create:

**Bug Report:**

- Something is broken or not working as expected
- Use bug template if available
- Include reproduction steps

**Feature Request:**

- New functionality or enhancement
- Use feature template if available
- Include use case and acceptance criteria

**Task/Chore:**

- Work item, refactoring, or maintenance
- Simple description of work needed
- Include definition of done

**Question/Discussion:**

- Need clarification or discussion
- May not have a clear resolution
- Use discussion if available, otherwise issue

### Step 2: Check for Existing Templates

```bash
# List available issue templates
gh api repos/:owner/:repo/contents/.github/ISSUE_TEMPLATE \
  --jq '.[].name' 2>/dev/null || echo "No templates found"

# Or check in root
ls -la .github/ISSUE_TEMPLATE/ 2>/dev/null
```

**If templates exist:**

```bash
# Create with specific template
gh issue create --template bug_report.md

# Or list templates interactively
gh issue create  # Will prompt for template
```

### Step 3: Gather Issue Information

**For Bug Reports:**

1. Title: Clear, specific description of the bug
2. Steps to reproduce
3. Expected behavior
4. Actual behavior
5. Environment details (OS, version, etc.)
6. Error messages or logs
7. Screenshots if relevant

**For Feature Requests:**

1. Title: Concise feature description
2. Problem/Use case: Why is this needed?
3. Proposed solution: How should it work?
4. Alternatives considered
5. Additional context

**For Tasks:**

1. Title: What needs to be done
2. Description: Context and details
3. Definition of done: How to know it's complete
4. Dependencies: Related issues or blockers

### Step 4: Create the Issue

**Simple issue:**

```bash
gh issue create \
  --title "Fix login button not responding" \
  --body "Button doesn't respond when clicked on mobile Safari"
```

**With template structure:**

```bash
gh issue create \
  --title "Bug: User authentication fails on Safari" \
  --body "$(cat <<'EOF'
## Description
Users cannot log in when using Safari browser on iOS.

## Steps to Reproduce
1. Open app in Safari on iPhone
2. Navigate to login page
3. Enter valid credentials
4. Click "Login" button
5. Nothing happens

## Expected Behavior
User should be logged in and redirected to dashboard.

## Actual Behavior
Login button doesn't respond. No error message shown.

## Environment
- Browser: Safari 17.2
- OS: iOS 17.3
- Device: iPhone 14 Pro
- App Version: 2.1.0

## Error Logs
```

Console: "Uncaught TypeError: Cannot read property 'submit'"

```

## Additional Context
Works fine on Chrome and Firefox desktop.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**With metadata:**

```bash
gh issue create \
  --title "Feature: Add dark mode support" \
  --body "..." \
  --label "enhancement,priority-high" \
  --assignee "@me" \
  --milestone "v2.0" \
  --project "Q1 Roadmap"
```

**Interactive mode:**

```bash
# Let gh prompt for all fields
gh issue create
```

**From file:**

```bash
gh issue create \
  --title "Bug: Memory leak in auth module" \
  --body-file bug-report.md \
  --label "bug,performance"
```

### Step 5: Link Related Items

**Link to PR:**

```bash
# Create issue and note PR number
gh issue create --title "..." --body "Related to #123"
```

**Reference other issues:**

```bash
gh issue create --title "..." --body "$(cat <<'EOF'
Depends on #45
Blocks #67
Related to #89
EOF
)"
```

### Step 6: Capture Issue URL

```bash
# Create and capture URL
ISSUE_URL=$(gh issue create --title "..." --body "..." 2>&1 | grep -o 'https://github.com/[^[:space:]]*')
echo "Created issue: $ISSUE_URL"

# Or get issue number
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')
echo "Issue #$ISSUE_NUM created"
```

### Step 7: Report to User

Present the result:

```markdown
✓ Issue #123 created successfully

Title: Fix login button on Safari
Type: Bug
Labels: bug, priority-high
Assigned: @username
Milestone: v2.0

🔗 [View Issue](https://github.com/owner/repo/issues/123)

Next steps:

- Start working: Use `gh-issue-develop` skill
- Add comments: Use `gh-issue-comment` skill
```

## Common Scenarios

### Scenario 1: Quick Bug Report

```bash
# Minimal bug report
gh issue create \
  --title "Bug: Checkout button crashes app" \
  --body "App crashes when user clicks checkout with empty cart" \
  --label "bug"
```

### Scenario 2: Detailed Feature Request

```bash
gh issue create \
  --title "Feature: Export data to CSV" \
  --body "$(cat <<'EOF'
## Problem
Users need to export their data for analysis in Excel/spreadsheets.

## Proposed Solution
Add "Export to CSV" button on the data table page that:
- Exports all visible columns
- Respects current filters
- Downloads immediately
- Includes headers

## Acceptance Criteria
- [ ] Export button added to data table toolbar
- [ ] CSV file includes all filtered rows
- [ ] Column headers are human-readable
- [ ] File named with timestamp: data_YYYY-MM-DD.csv
- [ ] Works for tables with >10,000 rows

## Alternatives Considered
- PDF export - Less flexible for analysis
- API endpoint - Too technical for end users
- Excel format - More complex, CSV is universal

## Additional Context
Users have been requesting this for 3 months.
Competitors already offer this feature.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
  --label "enhancement,user-request" \
  --assignee "alice" \
  --milestone "v2.1"
```

### Scenario 3: Task with Checklist

```bash
gh issue create \
  --title "Task: Update all dependencies" \
  --body "$(cat <<'EOF'
Update all npm dependencies to latest stable versions.

## Checklist
- [ ] Run `npm outdated` to identify updates
- [ ] Update minor/patch versions
- [ ] Test all functionality
- [ ] Update major versions one at a time
- [ ] Run full test suite after each major update
- [ ] Update lockfile
- [ ] Document breaking changes

## Definition of Done
- All dependencies on latest stable versions
- All tests passing
- No new security vulnerabilities
- CHANGELOG.md updated

Estimated time: 4 hours
EOF
)" \
  --label "maintenance,dependencies" \
  --assignee "@me"
```

### Scenario 4: Bulk Issue Creation from Audit

```bash
# Create multiple issues from a security audit report

while IFS=, read -r title severity description; do
  gh issue create \
    --title "Security: $title" \
    --body "$(cat <<EOF
## Severity
$severity

## Description
$description

## Remediation Required
Please address this security finding.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
    --label "security,$severity"
  sleep 1  # Rate limiting
done < security-findings.csv
```

### Scenario 5: Create Issue from Error Log

```bash
# Parse error logs and create issues

ERROR_MSG=$(grep "ERROR" app.log | tail -1)
STACK_TRACE=$(grep -A 10 "ERROR" app.log | tail -10)

gh issue create \
  --title "Bug: Application error - ${ERROR_MSG:0:50}" \
  --body "$(cat <<EOF
## Error Message
\`\`\`
$ERROR_MSG
\`\`\`

## Stack Trace
\`\`\`
$STACK_TRACE
\`\`\`

## Context
Occurred in production at $(date)
User ID: $USER_ID
Request ID: $REQUEST_ID

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
  --label "bug,production,needs-triage"
```

### Scenario 6: Create Issue from PR Review

```bash
# Create follow-up issues from PR review comments

gh issue create \
  --title "Refactor: Extract auth logic from handlers" \
  --body "$(cat <<'EOF'
Follow-up from PR #234 review.

## Context
During review of authentication PR, we identified that auth logic
is mixed with HTTP handlers, making it hard to test.

## Task
Extract authentication logic into separate service:
- Create `AuthService` class
- Move validation logic from handlers
- Add unit tests for `AuthService`
- Update handlers to use service

## Related
- PR #234
- Original implementation

Estimated time: 2-3 hours
EOF
)" \
  --label "refactoring,technical-debt" \
  --assignee "@me"
```

## Advanced Options

### Using Issue Templates

**Check available templates:**

```bash
gh api repos/:owner/:repo/contents/.github/ISSUE_TEMPLATE \
  | jq -r '.[].name'
```

**Create with specific template:**

```bash
gh issue create --template bug_report.yml
```

**Bypass template:**

```bash
gh issue create --template ""
```

### Issue Forms (GitHub Issue Forms)

For repositories using issue forms:

```bash
# List form fields
gh api repos/:owner/:repo/contents/.github/ISSUE_TEMPLATE/bug_report.yml \
  | jq -r '.content' | base64 -d

# Create issue (will prompt for form fields)
gh issue create --template bug_report.yml
```

### Projects and Milestones

**List available projects:**

```bash
gh project list
```

**List available milestones:**

```bash
gh api repos/:owner/:repo/milestones --jq '.[].title'
```

**Create with project:**

```bash
gh issue create \
  --title "..." \
  --body "..." \
  --project "Sprint 23"
```

### Auto-assign Based on Labels

```bash
# Auto-assign based on area
LABEL="backend"
case $LABEL in
  "backend")
    ASSIGNEE="bob"
    ;;
  "frontend")
    ASSIGNEE="alice"
    ;;
  "security")
    ASSIGNEE="charlie"
    ;;
esac

gh issue create \
  --title "..." \
  --body "..." \
  --label "$LABEL" \
  --assignee "$ASSIGNEE"
```

### Create Issue and Branch Together

```bash
# Create issue and immediately start working
ISSUE_URL=$(gh issue create --title "..." --body "...")
ISSUE_NUM=$(echo "$ISSUE_URL" | grep -o '[0-9]*$')

# Create branch for issue
gh issue develop $ISSUE_NUM --checkout
```

## Issue Templates

### Bug Report Template

```markdown
## Description

[Clear description of the bug]

## Steps to Reproduce

1. Step one
2. Step two
3. Step three

## Expected Behavior

[What should happen]

## Actual Behavior

[What actually happens]

## Environment

- OS: [e.g., macOS 14.0]
- Browser: [e.g., Chrome 120]
- Version: [e.g., 2.1.0]

## Error Messages
```

[Paste error messages here]

```

## Additional Context
[Any other relevant information]
```

### Feature Request Template

```markdown
## Problem Statement

[What problem does this solve?]

## Proposed Solution

[How should it work?]

## Use Cases

- Use case 1
- Use case 2

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Alternatives Considered

[Other approaches considered]

## Additional Context

[Mockups, examples, references]
```

## Tips

- **Use clear titles**: Make issues discoverable in searches
- **Add labels early**: Helps with triage and filtering
- **Assign ownership**: Clarify who's responsible
- **Link related items**: Connect issues and PRs
- **Use templates**: Ensure consistency and completeness
- **Include reproduction**: For bugs, always include steps to reproduce
- **Attach files**: Use `--body-file` for long descriptions
- **Set milestones**: Track issues by release
- **Use projects**: Organize work in project boards

## Error Handling

**Error: "Not authorized to create issues"**

- Cause: Insufficient repository permissions
- Solution: Request triage/write access or use personal fork

**Error: "Issue template not found"**

- Cause: Template doesn't exist
- Solution: List templates with `gh api` or create without template

**Error: "Invalid label"**

- Cause: Label doesn't exist in repository
- Solution: List labels with `gh label list` or create new label

**Error: "Invalid assignee"**

- Cause: User not a repository collaborator
- Solution: Add as collaborator or skip assignment

**Error: "Project not found"**

- Cause: Project doesn't exist or no access
- Solution: List projects with `gh project list`

## Best Practices

1. **Clear titles**: Use format "Type: Brief description"
   - Bug: Login fails on mobile
   - Feature: Add export to CSV
   - Task: Update dependencies

2. **Detailed descriptions**: Provide enough context for anyone to understand

3. **Use labels consistently**:
   - Type: bug, enhancement, task, question
   - Priority: priority-low, priority-high, critical
   - Area: backend, frontend, security, docs

4. **Assign appropriately**: Assign to person who will do the work, not reporter

5. **Link related items**: Use #123 syntax to reference issues/PRs

6. **Use milestones**: Track issues by release or sprint

7. **Include acceptance criteria**: Make it clear when issue is complete

8. **Attach evidence**: Add screenshots, logs, or reproduction repos

9. **Set expectations**: Estimate effort or timeline if known

10. **Follow up promptly**: Update issues as work progresses

## Related Skills

- `gh-issue-view` - View issue details
- `gh-issue-develop` - Start working on issue (create branch)
- `gh-issue-comment` - Add updates to issue
- `gh-issue-edit` - Update issue metadata
- `gh-issue-close` - Close/resolve issue

## Limitations

- Requires write/triage access to repository
- Cannot create issues in archived repositories
- Template selection limited in non-interactive mode
- Cannot attach binary files directly (must use separate upload)
- Project assignment may require additional permissions

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_create
- Issue templates: https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests
- Mastering Issues: https://docs.github.com/en/issues/tracking-your-work-with-issues
