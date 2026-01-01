---
name: gh-issue-comment
description: Add comments to GitHub issues using gh CLI. Provide updates, ask questions, share progress, or collaborate on issues. Use when user wants to communicate on an issue.
allowed-tools: Bash, Read, Grep
handoffs:
  - label: View Issue
    agent: gh-issue-view
    prompt: View this issue with all comments
    send: true
  - label: Close Issue
    agent: gh-issue-close
    prompt: Close this resolved issue
    send: true
---

# GitHub Issue Comment Skill

Add comments to GitHub issues using the `gh` CLI for collaboration and updates.

## When to Use

- User says "comment on issue #123" or "update the issue"
- Providing progress updates on work
- Asking questions about requirements
- Sharing findings or debugging info
- Notifying stakeholders of changes
- Adding test results or verification

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

## Execution Workflow

### Step 1: Verify Issue Exists

Check issue status before commenting:

```bash
# View issue
gh issue view 123 --json number,title,state

# Ensure it exists and is accessible
if [ $? -ne 0 ]; then
  echo "Error: Issue #123 not found"
  exit 1
fi
```

### Step 2: Determine Comment Type

Choose appropriate comment format:

**Progress Update:**

- Work status
- What's been done
- What's remaining
- Blockers or questions

**Question:**

- Request clarification
- Ask about requirements
- Seek input from team

**Resolution:**

- Announce fix
- Link to PR
- Explain solution

**Status Change:**

- Assign/unassign
- Label changes
- Milestone updates

### Step 3: Add Comment

**Simple comment:**

```bash
gh issue comment 123 --body "Working on this now. Will have a fix ready by EOD."
```

**Structured update:**

```bash
gh issue comment 123 --body "$(cat <<'EOF'
## Progress Update

**Completed:**
- ✅ Identified root cause
- ✅ Implemented fix
- ✅ Added unit tests

**In Progress:**
- 🔄 Testing on staging environment
- 🔄 Updating documentation

**Next Steps:**
- Create PR for review
- Deploy to production

ETA: Tomorrow morning

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**From file:**

```bash
gh issue comment 123 --body-file update.md
```

### Step 4: Verify Comment Posted

```bash
# View latest comments
gh issue view 123 --comments | tail -20

# Or check via JSON
gh issue view 123 --json comments \
  | jq -r '.comments[-1] | "\(.author.login): \(.body)"'
```

### Step 5: Report to User

```markdown
✓ Comment added to issue #123

Comment:

> Working on this now. Will have a fix ready by EOD.

🔗 [View Issue](https://github.com/owner/repo/issues/123)
```

## Common Scenarios

### Scenario 1: Progress Update

```bash
gh issue comment 123 --body "$(cat <<'EOF'
Quick update: I've reproduced the issue locally.

Root cause: Event listener not properly attached in Safari's strict mode.

Working on a fix now. Should have a PR ready this afternoon.
EOF
)"
```

### Scenario 2: Ask for Clarification

```bash
gh issue comment 123 --body "$(cat <<'EOF'
@alice Quick question about the requirements:

Should the export include *all* data or just the filtered view?

Also, what should happen if the dataset is >100K rows?

Thanks!
EOF
)"
```

### Scenario 3: Share Debug Information

```bash
ERROR_LOG=$(tail -50 /var/log/app.log)

gh issue comment 123 --body "$(cat <<EOF
Found the error in production logs:

\`\`\`
$ERROR_LOG
\`\`\`

Stack trace points to auth.ts:89. Investigating now.
EOF
)"
```

### Scenario 4: Link to PR

```bash
PR_URL=$(gh pr create --title "Fix login bug" --body "Fixes #123")

gh issue comment 123 \
  --body "Created PR $PR_URL to fix this issue. Ready for review!"
```

### Scenario 5: Share Test Results

```bash
TEST_OUTPUT=$(npm test | grep -A 20 "authentication")

gh issue comment 123 --body "$(cat <<EOF
Tested the fix across all browsers:

✅ Chrome 120 - PASS
✅ Firefox 121 - PASS
✅ Safari 17 - PASS
✅ Edge 119 - PASS

Test output:
\`\`\`
$TEST_OUTPUT
\`\`\`

Ready to merge!
EOF
)"
```

### Scenario 6: Notify About Deployment

```bash
gh issue comment 123 --body "$(cat <<'EOF'
🚀 **Deployed to Production**

Version: 2.1.0
Deployed: $(date)
Deployment ID: prod-2024-01-01-15-30

Verified working in production. Closing this issue.
EOF
)"

# Then close the issue
gh issue close 123 --comment "Verified fix in production. Issue resolved."
```

### Scenario 7: Add Checklist for Verification

```bash
gh issue comment 123 --body "$(cat <<'EOF'
Ready for testing! Please verify:

## Test Checklist
- [ ] Login works on Chrome
- [ ] Login works on Firefox
- [ ] Login works on Safari
- [ ] Login works on mobile
- [ ] Error handling works correctly
- [ ] Session persistence works

@alice @bob Can you help test?
EOF
)"
```

## Advanced Usage

### Comment with Mentions

```bash
# Notify specific users
gh issue comment 123 \
  --body "@alice @bob Please review the proposed solution above"
```

### Comment with Code Suggestions

````bash
gh issue comment 123 --body '$(cat <<'"'"'EOF'"'"'
Here's a potential fix:

```typescript
// Instead of:
addEventListener("click", handler);

// Use:
element.addEventListener("click", handler, false);
````

This ensures compatibility with Safari strict mode.
EOF
)'

````

### Add Comment from Template

```bash
# Create comment template
cat > comment-template.md <<'EOF'
## Update from @{user}

**Status:** {status}
**Progress:** {progress}%
**ETA:** {eta}

**Notes:**
{notes}

**Blockers:**
{blockers}
EOF

# Fill template
sed -e "s/{user}/$USER/" \
    -e "s/{status}/In Progress/" \
    -e "s/{progress}/75/" \
    -e "s/{eta}/Tomorrow/" \
    -e "s/{notes}/Almost done/" \
    -e "s/{blockers}/None/" \
    comment-template.md > filled-comment.md

# Post comment
gh issue comment 123 --body-file filled-comment.md
````

### Automated Status Comments

```bash
# Auto-comment when CI passes
gh pr view 234 --json statusCheckRollup \
  | jq -r '.statusCheckRollup[] | select(.conclusion == "SUCCESS")' \
  && gh issue comment 123 --body "✅ All CI checks passed for PR #234"
```

### Comment with Metrics

```bash
# Add performance metrics
gh issue comment 123 --body "$(cat <<EOF
## Performance Test Results

Before fix:
- Load time: 3.2s
- Memory: 145MB

After fix:
- Load time: 1.1s (66% improvement)
- Memory: 98MB (32% reduction)

Significant performance improvement! 🎉
EOF
)"
```

### Threading and Replies

```bash
# Get comment ID to reply to
COMMENT_ID=$(gh issue view 123 --json comments \
  | jq -r '.comments[] | select(.author.login == "alice") | .id' \
  | head -1)

# Reply to specific comment (requires API)
gh api repos/:owner/:repo/issues/comments/$COMMENT_ID/replies \
  -f body="@alice Good question! Here's my thinking..."
```

## Comment Best Practices

### Good Comment Structure

```markdown
## [Type]: Brief Title

**Context/Background:**

- Relevant information
- Links to related items

**Details:**

- Main content
- Code blocks
- Screenshots

**Action Items:**

- [ ] Task 1
- [ ] Task 2

**Questions/Blockers:**

- Any open questions
- Dependencies

@mentions if needed
```

### Effective Updates

```bash
gh issue comment 123 --body "$(cat <<'EOF'
## 🔄 Weekly Update

**This Week:**
- Completed authentication refactor
- Added 15 new tests
- Fixed 3 edge cases

**Next Week:**
- Code review
- Documentation update
- Deploy to staging

**Blockers:**
None currently

**Questions:**
@alice - Can you review the auth changes?
EOF
)"
```

## Tips

- **Be specific**: Include details, don't just say "working on it"
- **Use formatting**: Use headers, lists, code blocks for readability
- **Add context**: Link to related PRs, commits, or docs
- **Mention stakeholders**: Use @mentions to notify relevant people
- **Include evidence**: Add logs, screenshots, test results
- **Track progress**: Use checklists for multi-step work
- **Set expectations**: Include ETAs when appropriate
- **Ask clear questions**: Make it easy for others to help

## Error Handling

**Error: "Issue not found"**

- Cause: Issue doesn't exist or no access
- Solution: Verify issue number with `gh issue list`

**Error: "Not authorized"**

- Cause: Not authenticated or no repository access
- Solution: Run `gh auth login`

**Error: "Body required"**

- Cause: Empty comment body
- Solution: Provide comment text with `--body` or `--body-file`

**Error: "GraphQL error"**

- Cause: API rate limit or network issue
- Solution: Wait and retry

## Best Practices

1. **Update regularly**: Keep stakeholders informed of progress
2. **Be concise**: Get to the point quickly
3. **Format well**: Use markdown for readability
4. **Include evidence**: Add logs, screenshots, metrics
5. **Tag relevant people**: Use @mentions appropriately
6. **Link to context**: Reference PRs, commits, docs
7. **Use checklists**: Track multi-step work
8. **Set expectations**: Provide ETAs when possible
9. **Ask clear questions**: Make it easy to get answers
10. **Follow up**: Close the loop on questions and requests

## Comment Templates

### Progress Update Template

```markdown
## Progress Update

**Completed:**

- Item 1
- Item 2

**In Progress:**

- Item 3

**Next Steps:**

- Item 4

**Blockers:**
None

**ETA:** [date/time]
```

### Bug Investigation Template

```markdown
## Investigation Findings

**Root Cause:**
[Description]

**Affected Versions:**

- Version X
- Version Y

**Proposed Fix:**
[Solution approach]

**Testing Plan:**

- [ ] Test 1
- [ ] Test 2
```

### Question Template

```markdown
## Question

**Context:**
[Background information]

**Question:**
[Specific question]

**Why I'm Asking:**
[Rationale]

@[relevant-person] - Could you help with this?
```

## Related Skills

- `gh-issue-view` - View issue and comments
- `gh-issue-create` - Create new issues
- `gh-issue-edit` - Edit issue metadata
- `gh-issue-close` - Close resolved issues

## Limitations

- Cannot edit or delete comments via `gh issue comment` (use web UI or API)
- Cannot react to comments (use web UI)
- Limited formatting in CLI (no images, only markdown)
- Cannot create comment threads/replies easily via CLI

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_issue_comment
- Markdown guide: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
- Mentions: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#mentioning-people-and-teams
