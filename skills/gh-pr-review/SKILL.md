---
name: gh-pr-review
description: Review GitHub pull requests using gh CLI. Approve, request changes, or comment on PRs. Use when user wants to provide PR feedback or approval.
allowed-tools: Bash, Read, Grep, Glob
handoffs:
  - label: View PR
    agent: gh-pr-view
    prompt: View the PR details
    send: true
  - label: Merge PR
    agent: gh-pr-merge
    prompt: Merge this approved PR
    send: true
---

# GitHub PR Review Skill

Review pull requests using the `gh` CLI with comprehensive feedback options.

## When to Use

- User asks to "review the PR" or "approve PR #123"
- User wants to request changes on a PR
- User wants to leave comments on a PR
- Before merging, to provide approval
- User wants to review code changes in a PR

## Prerequisites

Verify GitHub CLI is installed and authenticated:

```bash
gh --version
gh auth status
```

## Execution Workflow

### Step 1: View PR for Review

First, examine the PR to review:

```bash
# View PR overview
gh pr view 123

# View diff
gh pr diff 123

# View specific files
gh pr diff 123 --patch | grep -A 50 "src/auth.ts"

# Check CI status
gh pr checks 123
```

### Step 2: Analyze Changes

Review the code:

1. **Check diff stats:**

```bash
gh pr diff 123 --stat
```

2. **Review commits:**

```bash
gh pr view 123 --json commits \
  | jq -r '.commits[] | "\(.messageHeadline)"'
```

3. **Check tests:**

```bash
gh pr diff 123 | grep -A 10 "\.test\."
```

4. **Look for issues:**
   - Code quality problems
   - Security vulnerabilities
   - Missing tests
   - Breaking changes
   - Style violations

### Step 3: Provide Review

Choose review type based on findings:

**APPROVE** - Code is good to merge

```bash
gh pr review 123 --approve
```

**REQUEST CHANGES** - Issues must be fixed

```bash
gh pr review 123 --request-changes \
  --body "Please address the following issues before merging."
```

**COMMENT** - Feedback without approval/rejection

```bash
gh pr review 123 --comment \
  --body "Looks good overall, minor suggestions below."
```

### Step 4: Add Review Comments

**General comment on PR:**

```bash
gh pr review 123 --approve --body "$(cat <<'EOF'
Great work! Code looks good overall.

Highlights:
- Clean implementation
- Good test coverage
- Follows team conventions

Suggestions:
- Consider extracting auth logic to separate module
- Add JSDoc for public API

Ready to merge! ✓
EOF
)"
```

**Comment on specific file/line:**

```bash
gh pr comment 123 \
  --body "Consider using const instead of let here" \
  --body-file review-comments.md
```

**Request changes with specific issues:**

```bash
gh pr review 123 --request-changes --body "$(cat <<'EOF'
Please address these issues:

## Security
- ⚠️ SQL injection vulnerability in user.ts:45
- ⚠️ Missing input validation in auth.ts:89

## Code Quality
- Consider extracting duplicate logic in handlers
- Missing error handling in payment.ts:123

## Tests
- Add test for edge case: empty user input
- Integration test needed for auth flow

Please update and re-request review.
EOF
)"
```

### Step 5: Follow Up

After review submitted:

```bash
# Verify review was posted
gh pr view 123 --json reviews \
  | jq -r '.reviews[-1] | "\(.author.login): \(.state)"'

# Check if PR can now be merged
gh pr view 123 --json reviewDecision
# APPROVED | CHANGES_REQUESTED | REVIEW_REQUIRED

# Monitor for updates
gh pr view 123 --comments
```

## Common Scenarios

### Scenario 1: Quick Approval

```bash
# Simple approval for straightforward PRs
gh pr view 123  # Quick review
gh pr checks 123  # Verify CI passed
gh pr review 123 --approve --body "LGTM! ✓"
```

### Scenario 2: Thorough Code Review

```bash
# Comprehensive review process

# 1. Get PR context
gh pr view 123 --json title,body,commits

# 2. Review all changes
gh pr diff 123 --patch > /tmp/pr-123.diff
# Review the diff file carefully

# 3. Check for common issues
gh pr diff 123 | grep -E "(TODO|FIXME|XXX|HACK)"
gh pr diff 123 | grep -E "(console\.log|debugger|print)"
gh pr diff 123 | grep -i "password.*=.*['\"]"

# 4. Verify tests exist
TEST_FILES=$(gh pr diff 123 --name-only | grep -c "\.test\.")
SRC_FILES=$(gh pr diff 123 --name-only | grep -c "\.ts$")
echo "Test coverage: $TEST_FILES tests for $SRC_FILES source files"

# 5. Check CI
gh pr checks 123

# 6. Provide detailed review
gh pr review 123 --approve --body "$(cat <<'EOF'
Excellent work! Comprehensive review completed.

✓ Code Quality: Clean, readable, follows conventions
✓ Security: No vulnerabilities found
✓ Tests: Good coverage (5 new tests)
✓ Performance: No concerns
✓ Documentation: Well documented

Ready to merge!
EOF
)"
```

### Scenario 3: Request Changes with Specific Feedback

````bash
# Found issues that need addressing

gh pr review 123 --request-changes --body "$(cat <<'EOF'
Good progress, but please address these issues:

## Critical Issues
1. **Security**: SQL injection in `src/db/user.ts:45`
   ```typescript
   // Current (vulnerable):
   db.query(`SELECT * FROM users WHERE id = ${userId}`)

   // Should be:
   db.query('SELECT * FROM users WHERE id = ?', [userId])
````

2. **Bug**: Race condition in `src/auth/session.ts:89`
   - Missing mutex/lock when updating session
   - Could lead to concurrent modification issues

## Non-Critical Improvements

- Consider extracting validation logic to separate file
- Add JSDoc comments for public API methods
- Update README with new auth flow

## Testing

- Missing test for expired token scenario
- Integration test needed for full auth flow

Please address critical issues and re-request review.
EOF
)"

````

### Scenario 4: Provide Comments Without Blocking

```bash
# Suggestions but not blocking merge

gh pr review 123 --comment --body "$(cat <<'EOF'
Nice work! A few optional suggestions:

**Performance Optimization:**
- Could cache the API response in `api.ts:123`
- Consider lazy loading the auth module

**Code Organization:**
- Might be cleaner to split `handlers.ts` into separate files
- Consider extracting constants to `config.ts`

These are just suggestions - feel free to merge as-is or address later!
EOF
)"
````

### Scenario 5: Review Multiple PRs in Batch

```bash
# Review all PRs from a specific author

gh pr list --author alice --json number,title \
  | jq -r '.[] | .number' \
  | while read pr; do
    echo "Reviewing PR #$pr..."

    # Quick check
    gh pr diff $pr --stat
    gh pr checks $pr

    # Auto-approve if small and tests pass
    CHANGES=$(gh pr diff $pr --stat | tail -1 | grep -oE '[0-9]+ insertions' | grep -oE '[0-9]+')
    CI_STATUS=$(gh pr checks $pr --json state --jq '.[].state' | sort -u)

    if [[ $CHANGES -lt 50 ]] && [[ "$CI_STATUS" == "SUCCESS" ]]; then
      gh pr review $pr --approve --body "Auto-approved: Small change, CI passed ✓"
      echo "✓ PR #$pr approved"
    else
      echo "⏭ PR #$pr requires manual review"
    fi
  done
```

### Scenario 6: Re-review After Changes

```bash
# Re-review after author addressed feedback

# Check what changed since last review
gh pr view 123 --json reviews,commits \
  | jq -r '
    .reviews[-1].submittedAt as $last_review |
    .commits[] |
    select(.committedDate > $last_review) |
    .messageHeadline
  '

# View new changes
gh pr diff 123

# Approve if issues addressed
gh pr review 123 --approve --body "$(cat <<'EOF'
Thanks for addressing the feedback!

✓ Security issues resolved
✓ Tests added
✓ Documentation updated

Approving now!
EOF
)"
```

## Advanced Review Techniques

### Automated Security Checks

```bash
# Security review automation

# Check for secrets/credentials
gh pr diff 123 | grep -iE "(api[_-]?key|password|secret|token|credential)"

# Check for debugging code
gh pr diff 123 | grep -E "(console\.log|debugger|print\()"

# Check for TODOs/FIXMEs
gh pr diff 123 | grep -E "(TODO|FIXME|XXX|HACK)"

# Check for unsafe patterns
gh pr diff 123 | grep -E "(eval\(|innerHTML|dangerouslySetInnerHTML)"

# Report findings
if [ $? -eq 0 ]; then
  gh pr review 123 --request-changes --body "Security concerns found - see comments"
fi
```

### Automated Code Quality Checks

```bash
# Run linters on PR changes
gh pr diff 123 --name-only | grep "\.ts$" | while read file; do
  npx eslint "$file" || echo "Lint errors in $file"
done

# Check test coverage
gh pr diff 123 --name-only | grep "\.test\." || \
  echo "⚠️ No test files in PR"

# Verify documentation
gh pr diff 123 --name-only | grep -q "README\|docs/" || \
  echo "⚠️ Consider updating documentation"
```

### Compare with Base Branch

```bash
# Check complexity increase
BASE_BRANCH=$(gh pr view 123 --json baseRefName --jq '.baseRefName')

# Lines of code delta
ADDITIONS=$(gh pr view 123 --json additions --jq '.additions')
DELETIONS=$(gh pr view 123 --json deletions --jq '.deletions')
NET_CHANGE=$((ADDITIONS - DELETIONS))

echo "Net change: +$NET_CHANGE lines"

if [ $NET_CHANGE -gt 500 ]; then
  echo "⚠️ Large PR - consider breaking into smaller pieces"
fi
```

### Review Checklist Automation

```bash
# Automated review checklist

PR_NUM=123
CHECKLIST=""

# 1. Check tests added
if gh pr diff $PR_NUM --name-only | grep -q "\.test\."; then
  CHECKLIST+="✓ Tests added\n"
else
  CHECKLIST+="⚠️ No tests found\n"
fi

# 2. Check CI status
if gh pr checks $PR_NUM | grep -q "✓"; then
  CHECKLIST+="✓ CI passed\n"
else
  CHECKLIST+="❌ CI failing\n"
fi

# 3. Check size
CHANGES=$(gh pr view $PR_NUM --json additions,deletions --jq '.additions + .deletions')
if [ $CHANGES -lt 300 ]; then
  CHECKLIST+="✓ Reasonable size ($CHANGES lines)\n"
else
  CHECKLIST+="⚠️ Large PR ($CHANGES lines)\n"
fi

# 4. Check documentation
if gh pr diff $PR_NUM --name-only | grep -qE "(README|docs/)"; then
  CHECKLIST+="✓ Documentation updated\n"
else
  CHECKLIST+="ℹ️ No documentation changes\n"
fi

# Post checklist as review
gh pr review $PR_NUM --comment --body "$(echo -e "Review Checklist:\n\n$CHECKLIST")"
```

## Tips

- **Review promptly**: Don't let PRs sit for days
- **Be constructive**: Suggest improvements, don't just criticize
- **Test locally**: For complex PRs, check out and test locally
- **Check tests**: Always verify tests are included and passing
- **Security first**: Look for security vulnerabilities
- **Ask questions**: Use comments to clarify unclear code
- **Approve explicitly**: Don't assume - explicitly approve when ready
- **Use templates**: Create review templates for consistency

## Error Handling

**Error: "Not authorized to review"**

- Cause: Insufficient permissions or reviewing your own PR
- Solution: Verify repository access or ask another reviewer

**Error: "Pull request is not open"**

- Cause: PR is closed or merged
- Solution: Verify PR status with `gh pr view 123`

**Error: "Review already submitted"**

- Cause: You already reviewed this version
- Solution: Wait for new commits or edit existing review

**Error: "Cannot approve your own PR"**

- Cause: GitHub prevents self-approval
- Solution: Request review from teammates

## Best Practices

1. **Review completely**: Check code, tests, docs, CI
2. **Be specific**: Point to exact lines/files with issues
3. **Suggest solutions**: Don't just point out problems
4. **Approve explicitly**: When ready, explicitly approve
5. **Request changes clearly**: List all issues that must be fixed
6. **Use comments for suggestions**: Don't block merge for minor issues
7. **Test complex changes**: Check out PR locally for thorough testing
8. **Check security**: Always look for security vulnerabilities
9. **Verify tests**: Ensure adequate test coverage
10. **Be timely**: Review within 24 hours when possible

## Review Guidelines

### When to APPROVE

- Code is high quality
- Tests are comprehensive
- No security issues
- Follows team conventions
- CI passes
- Documentation adequate

### When to REQUEST CHANGES

- Security vulnerabilities
- Critical bugs
- Missing tests for new features
- Breaking changes without migration
- Violates team standards
- CI failing

### When to COMMENT

- Minor suggestions
- Style preferences
- Performance optimizations
- Future improvements
- Questions about approach
- Positive feedback

## Related Skills

- `gh-pr-view` - View PR details before reviewing
- `gh-pr-merge` - Merge approved PRs
- `gh-pr-create` - Create new PRs
- `gh-pr-ready` - Mark draft as ready

## Limitations

- Cannot review your own PRs
- Limited to text-based review (no visual diff in CLI)
- Cannot suggest specific code changes (use web UI for suggestions)
- No threading support for conversations
- Cannot resolve conversations via CLI

## See Also

- GitHub CLI docs: https://cli.github.com/manual/gh_pr_review
- Code review best practices: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests
- Review comments: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/commenting-on-a-pull-request
