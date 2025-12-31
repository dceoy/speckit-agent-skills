---
name: gemini-review
description: Perform comprehensive code reviews using Google Gemini CLI to identify bugs, security vulnerabilities, performance issues, and code quality problems. Use after code changes, before commits, during pull requests, or when security/quality validation is needed. Requires Gemini CLI installed.
allowed-tools: Bash, Read, Grep, Glob
---

# Gemini Review Skill

Use Google Gemini CLI to perform thorough code reviews identifying bugs, security issues, performance problems, and quality improvements. This is a **read-only review** skill with multimodal capabilities.

## When to Use

- User wants code reviewed before committing
- User needs security vulnerability assessment
- User wants performance optimization suggestions
- User requests code quality improvements
- User needs design compliance verification
- User wants to review pull requests
- User needs pre-deployment validation
- User wants to compare implementation with design specs

## Prerequisites

Verify Gemini CLI is available:

```bash
gemini --version  # Should display installed version
```

Ensure authentication is configured (OAuth or API key).

## Basic Usage

### Step 1: Determine Scope

Identify what to review:

- **Uncommitted changes** (default): `git diff`
- **Last commit**: `git diff HEAD~1`
- **Pull request**: `git diff main...branch`
- **Specific files**: User-specified files
- **Entire codebase**: Full project review
- **Design compliance**: Compare with mockups/specs

### Step 2: Gather Context

Before reviewing:

```bash
# Check what changed
git status
git diff --stat
git diff

# Check project context
cat GEMINI.md  # if exists

# Check for existing issues
npm run lint 2>&1 | head -20
npm run typecheck 2>&1 | head -20
```

### Step 3: Execute Gemini Review

**Comprehensive review:**

```bash
gemini --include-directories src,lib,tests -p "Perform a comprehensive code review of changes in git diff.

Focus on:

1. CRITICAL ISSUES (must fix immediately):
   - Security vulnerabilities (SQL injection, XSS, CSRF)
   - Potential runtime errors
   - Data loss risks
   - Breaking changes

2. IMPORTANT ISSUES (should fix soon):
   - Logic bugs
   - Performance problems
   - Type safety gaps
   - Error handling issues
   - Resource leaks

3. SUGGESTIONS (consider improving):
   - Code quality improvements
   - Refactoring opportunities
   - Better patterns
   - Documentation needs

4. POSITIVE OBSERVATIONS:
   - Best practices followed
   - Good patterns used

For each issue provide:
- Severity: Critical | Important | Suggestion
- File path and line number
- Clear description
- Why it's a problem
- How to fix it
- Code example of the fix

Use Google Search to verify against current OWASP standards.
Do NOT make any changes - this is review only."
```

**Quick pre-commit scan:**

```bash
gemini -p "Quick pre-commit review of git diff:
- console.log or debug statements
- Unused imports
- TODO/FIXME comments
- Missing error handling
- Obvious type errors
- Hardcoded secrets"
```

**Security-focused review:**

```bash
gemini -p "Security-focused review of git diff:
- SQL injection vulnerabilities
- XSS vulnerabilities
- CSRF vulnerabilities
- Authentication/authorization flaws
- Secrets in code
- Input validation gaps
- Session management issues
- OWASP Top 10 risks

Use Google Search to verify against current OWASP 2026 recommendations."
```

**Performance review:**

```bash
gemini --include-directories src -p "Performance review of git diff:
- Inefficient algorithms
- Unnecessary re-renders
- Memory leaks
- N+1 queries
- Blocking operations
- Missing caching
- Unoptimized assets"
```

**Design compliance review (multimodal):**

```bash
gemini --include-files design-spec.pdf,mockup.png -p "Review code changes for compliance with design:
- Compare implementation with mockup.png
- Verify requirements from design-spec.pdf are met
- Check for visual/functional discrepancies
- Validate responsive behavior
- Ensure accessibility requirements followed"
```

### Step 4: Present Review

Format results clearly:

````markdown
# Code Review: [Scope]

## Summary

- Files reviewed: X
- Issues found: Y (Critical: A, Important: B, Suggestions: C)
- Overall assessment: [Brief verdict]

---

## 🔴 Critical Issues (Fix Immediately)

### src/auth/login.ts:45 - SQL Injection Vulnerability

**Severity:** Critical
**Category:** Security

**Problem:**
User input is directly concatenated into SQL query without sanitization.

**Why it matters:**
Allows attackers to execute arbitrary SQL commands, potentially exposing all user data.

**How to fix:**

1. Use parameterized queries
2. Validate input
3. Escape special characters

**Code example:**

```typescript
// Before (problematic)
const query = `SELECT * FROM users WHERE email = '${email}'`;

// After (fixed)
const query = "SELECT * FROM users WHERE email = ?";
const result = await db.execute(query, [email]);
```

**Reference:**
[OWASP SQL Injection Prevention](https://url-from-google-search)

---

## 🟡 Important Issues (Should Fix)

[Same format]

---

## 🟢 Suggestions (Consider Improving)

[Same format]

---

## ✅ Positive Observations

- `auth.ts:89` - Proper input validation and sanitization
- `utils.ts:45` - Good use of memoization

---

## Recommended Actions

1. **Immediate:** Fix SQL injection in auth.ts:45
2. **Soon:** Address N+1 query in user service
3. **Consider:** Refactor large function

---

## Sources

[If Google Search grounding was used, list URLs]
````

## Example Reviews

### Pre-Commit Review

```bash
gemini -p "Quick pre-commit review for obvious issues:
- No debug code
- No commented-out code
- All imports used
- No TODO comments without issues
- Error handling present
- Types complete
- No hardcoded secrets"
```

### Pull Request Review

```bash
gemini -p "Comprehensive PR review of all changes vs main:
- Code quality and standards
- Security vulnerabilities (verify with Google Search)
- Performance implications
- Breaking changes
- Test coverage
- Documentation updates
- API contract changes
- Backward compatibility"
```

### Pre-Deployment Review

```bash
gemini -p "Pre-deployment security and stability review:
- No secrets in code
- Environment configs externalized
- Error logging implemented
- Security vulnerabilities addressed (check OWASP Top 10)
- Database migrations are safe
- Rollback plan exists
- Feature flags for risky changes"
```

### Test Coverage Review

```bash
gemini --include-directories src,tests -p "Test coverage review:
- Untested code paths
- Missing edge cases
- Brittle tests
- Incomplete assertions
- Missing integration tests
- Flaky tests"
```

### Design Compliance Review

```bash
gemini --include-files design.pdf,mockup-1.png,mockup-2.png -p "Review implementation against design:
- Visual accuracy vs mockups
- All requirements from design.pdf implemented
- Responsive behavior matches design
- Accessibility requirements met
- Colors, typography, spacing match design system
- User interactions match specified behavior"
```

## Gemini-Specific Features

### Google Search Grounding

Verify against current standards:

```bash
gemini -p "Review authentication implementation. Use Google Search to verify it follows current OWASP 2026 authentication security standards."
```

### Multimodal Review

Compare with design artifacts:

```bash
gemini --include-files original-design.png -p "Review Dashboard component vs original-design.png:
- Visual fidelity
- Implementation of all design elements
- Responsive behavior
- Accessibility features
- Performance considerations"
```

### Large Context Window

Review entire projects:

```bash
gemini --include-directories src,lib,tests,config -p "Comprehensive review of entire application:
- Architecture patterns
- Security across all modules
- Performance bottlenecks
- Code quality standards
- Test coverage"
```

### Conversation Checkpointing

Multi-phase review:

```bash
# Phase 1: Security
gemini -p "First, review for security issues only"

# Phase 2: Performance (context preserved)
gemini -p "Now review the same changes for performance issues"

# Phase 3: Code quality
gemini -p "Finally, review for code quality and maintainability"
```

## Best Practices

✅ **DO:**

- Verify findings before reporting (use Read tool)
- Prioritize by severity
- Provide specific file paths and line numbers
- Suggest concrete fixes with code examples
- Balance critique with positive observations
- Use Google Search to verify against current standards
- Leverage multimodal for design compliance
- Be constructive and professional

❌ **DON'T:**

- Modify code during review
- Report false positives
- Use vague language
- Skip verification
- Focus only on negatives
- Ignore positive practices

## Verification Steps

Before presenting review:

- [ ] All file paths are correct
- [ ] Line numbers are accurate (verify with Read)
- [ ] Issues are real (not false positives)
- [ ] Severity levels appropriate
- [ ] Fix suggestions are helpful
- [ ] Positive observations included
- [ ] Review is constructive

## Error Handling

**If Gemini not found:**

```
Gemini CLI is not available in PATH. Ensure it is installed per the prerequisites in README.md.
```

**If authentication fails:**

```
Gemini CLI needs authentication. Run:
gemini

Then sign in with Google account or configure API key.
```

**If review is unclear:**

- Re-run with more specific focus
- Break into smaller scopes
- Use `--include-directories` to narrow context

**If too many issues found:**

- Prioritize by severity
- Group related issues
- Create separate reviews per category
- Fix critical first, then iterate

**If false positives occur:**

- Verify each issue with Read tool
- Filter out non-issues
- Refine prompts

**If context window exceeded:**

```bash
# Review in focused chunks
gemini --include-directories src/auth -p "Review auth code only"
gemini --include-directories src/api -p "Review API code only"
```

## Review Checklist Template

```
Code Quality:
- [ ] Follows coding standards
- [ ] No code duplication
- [ ] Functions are focused
- [ ] Clear names
- [ ] Comments explain "why"

Security:
- [ ] No SQL injection
- [ ] No XSS vulnerabilities
- [ ] Input validation present
- [ ] No secrets in code
- [ ] Auth/authz correct
- [ ] Follows OWASP standards

Performance:
- [ ] Algorithms efficient
- [ ] No unnecessary operations
- [ ] Appropriate caching
- [ ] No memory leaks

Testing:
- [ ] Tests added/updated
- [ ] Edge cases covered
- [ ] Error cases tested
- [ ] Good coverage

Accessibility:
- [ ] Semantic HTML
- [ ] ARIA attributes
- [ ] Keyboard navigation
- [ ] Screen reader compatible

Documentation:
- [ ] README updated
- [ ] API docs current
- [ ] Complex logic documented
```

## Related Skills

- **gemini-ask**: For understanding code before review
- **gemini-exec**: For implementing fixes after review
- **gemini-search**: For researching current standards

## Tips for Better Results

1. **Be specific** about focus areas
2. **Use Google Search** to verify against current standards
3. **Include designs** with `--include-files` for compliance checks
4. **Narrow scope** with `--include-directories` for large codebases
5. **Multi-phase** reviews for comprehensive analysis
6. **Verify findings** with Read tool before presenting

## Advanced Usage

### Comparative Review

```bash
gemini --include-files spec-v1.pdf,spec-v2.pdf -p "Review implementation and explain:
- Which spec version is implemented
- Discrepancies from both versions
- Missing features from target spec"
```

### Architecture Review

```bash
gemini --include-files architecture-diagram.png --include-directories src -p "Review architecture implementation:
- Compare code structure with diagram
- Identify architectural violations
- Suggest improvements for alignment
- Check for anti-patterns"
```

### Cross-Module Review

```bash
gemini --include-directories src/frontend,src/backend -p "Review integration between frontend and backend:
- API contract consistency
- Type safety across boundaries
- Error handling on both sides
- Security at integration points"
```

## Limitations

- Cannot modify code
- Read-only static analysis
- May need manual verification of findings
- Quality depends on context provided
- Some issues require runtime analysis

## Unique Capabilities vs Other Tools

**vs codex-review:**

- ✅ Multimodal review (compare with designs)
- ✅ Built-in Google Search grounding
- ✅ 1M token context window
- ✅ Conversation checkpointing
- ✅ Better for design compliance checks

**vs copilot-review:**

- ✅ Multimodal capabilities
- ✅ Google Search integration
- ✅ Larger context window
- ✅ Better for visual compliance

---

**Remember**: This skill is READ-ONLY. Analyze and suggest, never modify. Use Google Search and multimodal capabilities for comprehensive, current feedback.
