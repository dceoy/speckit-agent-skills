---
name: copilot-review
description: Perform code reviews using GitHub Copilot CLI to identify bugs, security vulnerabilities, performance issues, and code quality problems. Use when the user asks to review code, check for issues, security audit, or before committing. Requires Copilot CLI installed.
allowed-tools: Bash, Read, Grep, Glob
---

# Copilot Review Skill

Use GitHub Copilot CLI to perform automated code reviews that identify issues and suggest improvements. This is a **read-only** analysis skill.

## When to Use

- User asks to "review" code
- User wants to check for bugs or issues
- User mentions "security", "performance", or "quality"
- Before committing code
- During pull request review
- User asks "what's wrong with this code?"

## Prerequisites

Verify GitHub Copilot CLI is available:

```bash
copilot --version  # Should display installed version
```

Or check if the command exists:

```bash
which copilot
```

### Authentication

Ensure you're authenticated:

```bash
copilot
# Then run: /login
# Follow on-screen instructions to authenticate with GitHub
```

**Requirements:**

- Active GitHub Copilot subscription
- Node.js v22+ (for Copilot CLI itself)

## Basic Usage

### Step 1: Determine Scope

What to review:

- Uncommitted changes (default)
- Specific file(s)
- Last commit
- Pull request
- Entire codebase

### Step 2: Check Current State

```bash
git status          # See what's changed
git diff --stat     # Summary of changes
git diff            # Detailed changes
```

### Step 3: Launch Copilot CLI

Navigate to the project and launch:

```bash
cd /path/to/project
copilot
```

### Step 4: Execute Review

Provide a comprehensive review prompt:

```
Perform comprehensive code review of [SCOPE].

Check for:
1. CRITICAL ISSUES (must fix):
   - Security vulnerabilities (SQL injection, XSS, CSRF, etc.)
   - Potential runtime errors
   - Data loss risks
   - Breaking changes

2. IMPORTANT ISSUES (should fix):
   - Logic bugs
   - Performance problems
   - Type safety gaps
   - Error handling issues

3. SUGGESTIONS (consider):
   - Code quality improvements
   - Refactoring opportunities
   - Better patterns
   - Documentation needs

4. POSITIVE OBSERVATIONS:
   - Best practices followed
   - Good patterns used

For each issue:
- Severity level (Critical/Important/Suggestion)
- File path and line number
- Clear description of the problem
- Why it's a problem
- How to fix it

Do NOT make any changes - this is review only.
```

### Step 5: Present Findings

Organize results by severity:

- 🔴 Critical Issues
- 🟡 Important Issues
- 🟢 Suggestions
- ✅ Positive Observations

## Example Reviews

### Review Uncommitted Changes

```
Review all uncommitted changes for:
- Bugs and logic errors
- Security vulnerabilities
- Performance issues
- Code quality problems
- Missing error handling

Do NOT modify code.
```

### Security-Focused Review

```
Security review of src/auth/*.ts:
- SQL injection vulnerabilities
- XSS vulnerabilities
- Authentication bypass
- Authorization flaws
- Secrets in code
- Input validation gaps

Provide severity level and fix suggestions. Do NOT modify code.
```

### Performance Review

```
Performance review of src/components/*.tsx:
- Unnecessary re-renders
- Missing React.memo, useMemo, useCallback
- Inefficient algorithms
- Memory leaks
- Large bundle impacts

Provide specific optimization suggestions. Do NOT modify code.
```

### Pre-Commit Review

```
Quick review of staged changes for:
- console.log statements
- Commented-out code
- Unused imports
- TODO comments
- Missing error handling
- Type errors

Do NOT modify code.
```

### Pull Request Review

```
Review PR #[NUMBER] for:
- Code quality and best practices
- Security issues
- Performance concerns
- Test coverage
- Documentation completeness

Do NOT modify code.
```

## Review Focus Areas

### General Review

```
Comprehensive review covering: security, performance, code quality, architecture, testing, accessibility.

Do NOT modify code.
```

### Security Audit

```
Security audit focusing on: SQL injection, XSS, CSRF, authentication, authorization, secrets, input validation, OWASP Top 10.

Do NOT modify code.
```

### Architecture Review

```
Architecture review: SOLID principles, separation of concerns, dependency management, code organization, design patterns.

Do NOT modify code.
```

### Accessibility Review

```
Accessibility review: ARIA labels, keyboard navigation, screen reader support, color contrast, semantic HTML, WCAG compliance.

Do NOT modify code.
```

## Output Format

Structure review results:

````markdown
# Code Review: [Scope]

## Summary

- Files reviewed: 3
- Issues found: 5 (Critical: 1, Important: 2, Suggestions: 2)
- Estimated fix time: 2 hours

## 🔴 Critical Issues (Fix Immediately)

### src/auth/login.ts:45 - SQL Injection Vulnerability

**Severity:** Critical
**Category:** Security

**Problem:**
Direct string interpolation in SQL query allows SQL injection.

**Why it matters:**
Attacker can execute arbitrary SQL commands, steal data, or drop tables.

**How to fix:**
Use parameterized queries:

```typescript
// Before (vulnerable)
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// After (safe)
db.query("SELECT * FROM users WHERE email = ?", [email]);
```

---

## 🟡 Important Issues (Should Fix)

[Same format as critical]

---

## 🟢 Suggestions (Consider Improving)

[Same format]

---

## ✅ Positive Observations

- src/utils/validation.ts:23 - Excellent input sanitization
- src/hooks/useAuth.ts:67 - Proper cleanup in useEffect

---

## Recommended Actions

1. **Immediate:** Fix SQL injection in auth/login.ts:45
2. **Soon:** Address performance issue in components/UserList.tsx
3. **Consider:** Refactor large function at utils/helpers.ts:120
````

## Best Practices

✅ **DO:**

- Categorize by severity (Critical/Important/Suggestion)
- Include specific file paths and line numbers
- Explain WHY something is a problem
- Provide clear fix suggestions
- Note positive observations too
- Verify findings before reporting

❌ **DON'T:**

- Make code modifications (use copilot-exec for that)
- Skip verification of findings
- Report false positives without investigation
- Be purely negative without noting good practices

## Model Selection

GitHub Copilot CLI defaults to Claude Sonnet 4.5 but supports other models:

```
/model
```

Then choose from:

- Claude Sonnet 4.5 (default, excellent for reviews)
- Claude Sonnet 4
- GPT-5

## Verification

After getting Copilot's review:

1. Verify file paths and line numbers are correct
2. Check if issues are real (not false positives)
3. Assess severity appropriately
4. Add context from your knowledge of the code

## Error Handling

**If Copilot not found:**

```
GitHub Copilot CLI is not available.

Ensure it's installed and available in your PATH:
- Check: which copilot
- Verify: copilot --version
```

**If too many issues:**

- Focus on critical issues first
- Group related issues
- Break into multiple focused reviews

**If false positives:**

- Manually verify each issue
- Filter out non-issues
- Clarify with more specific review scope

## GitHub Integration

Copilot CLI can review GitHub PRs directly:

```
Review the changes in pull request #123
```

```
What issues exist in issue #456?
```

This provides access to:

- PR diff
- Issue description
- Comments and discussion
- Linked commits

## Review Checklist Templates

### General Checklist

```
- [ ] No console.log or debug statements
- [ ] No commented-out code
- [ ] All imports are used
- [ ] No TODO comments (or tracked in issues)
- [ ] Error handling present
- [ ] Types complete
- [ ] No security vulnerabilities
- [ ] Tests pass
```

### Security Checklist

```
- [ ] No SQL injection risks
- [ ] No XSS vulnerabilities
- [ ] Input validation present
- [ ] No secrets in code
- [ ] Authentication/authorization correct
- [ ] HTTPS enforced
- [ ] CSRF protection enabled
```

## Quota Management

Each prompt uses one premium request from your monthly quota:

- Be thorough in your review request
- Combine multiple concerns into one prompt
- Focus on specific areas when quota is limited
- See GitHub docs for quota limits

## Related Skills

- **copilot-ask**: For understanding code before reviewing
- **copilot-exec**: For fixing issues found in review

## Tips for Better Reviews

1. **Be specific**: "Review src/auth.ts for security" vs "Review code"
2. **Define scope**: Specific files > entire codebase
3. **Choose focus**: Security audit vs general review
4. **Iterate**: Review → Fix → Re-review
5. **Combine skills**: copilot-ask → copilot-review → copilot-exec

## Interactive Workflow

Since Copilot CLI is interactive:

1. **Launch**: `copilot` in project directory
2. **Describe**: Explain what to review and focus areas
3. **Review**: Copilot analyzes and provides feedback
4. **Clarify**: Ask follow-up questions
5. **Document**: Save findings for the team
6. **Exit**: Use Ctrl+C or exit command

## Advanced Review Scenarios

### Diff-Based Review

```
Review the diff between main and feature-branch for:
- Breaking changes
- Migration concerns
- Documentation updates needed
```

### Security Compliance

```
Review for SOC 2 compliance:
- Audit logging
- Data encryption
- Access controls
- PII handling
```

### Performance Optimization

```
Identify performance bottlenecks in:
- Database queries (N+1 problems)
- Rendering performance
- Memory usage
- Network requests
```

## Limitations

- Interactive mode only (no direct scripting)
- Static analysis only (cannot run code)
- May generate false positives
- Cannot understand business logic
- Cannot test runtime behavior
- Limited by context window size
- Uses quota per prompt

## Integration with Claude Code

When using this skill from Claude Code:

1. Check git status to understand scope
2. Guide users to open a terminal
3. Navigate to the project directory
4. Launch `copilot`
5. Provide the formatted review request
6. Users interact with Copilot to review findings
7. Summarize key findings and recommendations
8. Create action items for critical issues

---

**Remember**: This skill is READ-ONLY. To fix issues found, use the `copilot-exec` skill.
