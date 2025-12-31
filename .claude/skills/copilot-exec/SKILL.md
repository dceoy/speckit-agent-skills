---
name: copilot-exec
description: Execute development tasks using GitHub Copilot CLI for code generation, refactoring, feature implementation, and bug fixes. Use when the user asks to create code, add features, refactor, fix bugs, or generate tests. Requires Copilot CLI installed.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Copilot Exec Skill

Use GitHub Copilot CLI to execute development tasks that involve code modifications. This skill **modifies code**.

## When to Use

- User asks to "add", "create", "implement", or "generate" code
- User wants to refactor existing code
- User needs to fix a bug
- User wants to create tests
- User asks to "update" or "modify" code

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

### Step 1: Understand the Task

Parse what needs to be done:

- What to create/modify?
- Which files are affected?
- What's the expected outcome?
- Any constraints or requirements?

### Step 2: Gather Context

Before executing, understand current state:

```bash
git status  # Check for uncommitted changes
git diff    # See existing modifications
```

Read relevant files to understand patterns.

### Step 3: Launch Copilot CLI

Navigate to the project and launch:

```bash
cd /path/to/project
copilot
```

### Step 4: Execute Task

Provide clear, structured instructions to Copilot:

```
[TASK DESCRIPTION]

Follow these guidelines:
- Follow existing code patterns and conventions
- Add appropriate error handling
- Include necessary imports
- Maintain code quality and readability
- Use proper types (TypeScript/other)
- Add comments for complex logic

Project context:
- Language: [e.g., TypeScript, Python, Go, Rust]
- Framework: [e.g., React, Django, Express]
- Build tool: [e.g., Vite, webpack, cargo, go build]
- Coding style: [Reference style guide or linter config]

Preview all changes before applying them.
```

### Step 5: Preview and Approve

Copilot CLI requires explicit approval before making changes:

- Review each proposed change carefully
- Approve changes that look correct
- Reject or modify changes that need adjustment

### Step 6: Verify Changes

After execution:

```bash
git status              # See what changed
git diff                # Review modifications
<lint-command>          # Check for errors
<test-command>          # Run tests
```

### Step 7: Report Results

Provide clear summary:

- What files were modified/created
- What changes were made
- Verification results (lint, types, tests)
- Any issues or warnings

## Example Tasks

### Generate New Code

```
Create a UserProfile component in src/components/ with:
- Props: name (string), email (string), avatar (string optional)
- Display user info in a card layout
- Use TypeScript types
- Follow existing component patterns
- Add CSS modules for styling
```

### Refactor Code

```
Refactor the validation logic in src/components/LoginForm.tsx:
- Extract validation into separate src/utils/validation.ts file
- Create validateEmail and validatePassword functions
- Maintain all existing functionality
- Add TypeScript types
```

### Fix Bugs

```
Fix the memory leak in src/hooks/useWebSocket.ts:
- The WebSocket connection is not being cleaned up
- Add proper cleanup in useEffect return function
- Ensure connection closes on unmount
```

### Add Features

```
Add email validation to ContactForm.tsx:
- Use regex: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
- Show error message below input field
- Validate on blur and submit
- Prevent submission if invalid
- Style error message in red
```

### Create Tests

```
Create unit tests for src/utils/validation.ts:
- Test validateEmail with valid/invalid inputs
- Test validatePassword with edge cases
- Test error messages
- Use Jest and React Testing Library
- Aim for 100% coverage
```

## Verification Workflow

After Copilot executes, ALWAYS:

1. **Review changes**:

   ```bash
   git diff
   ```

2. **Run linter**:

   ```bash
   <lint-command>
   ```

3. **Run tests**:

   ```bash
   <test-command>
   ```

4. **Manual testing**:

   Start your development server and test the functionality manually.

5. **Report status**:
   - ✅ Changes applied successfully
   - ✅ Linter passed
   - ✅ Tests passed
   - ✅ Manual testing confirmed

## Best Practices

✅ **DO:**

- Be specific in task descriptions
- Include file paths
- Specify expected behavior
- Review all changes with `git diff`
- Run linter and tests
- Test manually before declaring success
- Use the preview feature before approving changes

❌ **DON'T:**

- Use vague instructions like "make it better"
- Skip verification steps
- Approve changes without reviewing
- Commit without verifying
- Ignore errors or warnings

## Model Selection

GitHub Copilot CLI defaults to Claude Sonnet 4.5 but supports other models:

```
/model
```

Then choose from:

- Claude Sonnet 4.5 (default, recommended for code tasks)
- Claude Sonnet 4
- GPT-5

Different models may excel at different types of tasks.

## Error Handling

**If execution fails:**

1. Check the error message from Copilot
2. Verify authentication is valid
3. Try with more specific instructions
4. Break task into smaller steps

**If changes are incorrect:**

1. Revert: `git restore .` or `git restore <file>`
2. Re-execute with better instructions
3. Or fix manually

**If tests fail:**

1. Review what broke
2. Ask Copilot to fix the test failures
3. Or fix manually

## Writing Effective Tasks

### Good Task Description

```
Add email validation to ContactForm.tsx using regex pattern.
Show error message below the input field on blur.
Prevent form submission if email is invalid.
Style the error message in red (#dc2626).
Follow existing form validation patterns.
```

### Poor Task Description

```
"Add validation"  # Too vague
"Fix the form"    # No specifics
```

## Safety Checklist

Before using copilot-exec:

- [ ] Understand what will be modified
- [ ] Have clean git state (can rollback)
- [ ] Know which files will be affected
- [ ] Have tests to verify correctness

After using copilot-exec:

- [ ] Reviewed changes with `git diff`
- [ ] Ran linter (language-specific)
- [ ] Ran type checker (if applicable)
- [ ] Ran tests (language-specific)
- [ ] Tested manually
- [ ] Ready to commit

## GitHub Integration

Copilot CLI can access GitHub context:

```
Create a new feature based on issue #123
```

```
Implement the changes requested in PR #456
```

## MCP Server Integration

If you have custom MCP (Model Context Protocol) servers configured, Copilot CLI can use them:

- Configure MCP servers in your Copilot settings
- Reference custom tools or data in your prompts
- Extend capabilities beyond code editing

## Quota Management

Each prompt uses one premium request from your monthly quota:

- Combine related changes into one task when possible
- Be thorough in your initial request
- Use follow-up questions sparingly
- See GitHub docs for quota limits

## Related Skills

- **copilot-ask**: For understanding code before modifying
- **copilot-review**: For reviewing changes after modification

## Tips for Success

1. **Start small**: Test with simple tasks first
2. **Be specific**: Include file paths, requirements, constraints
3. **Follow patterns**: Reference existing code to follow
4. **Verify always**: Never skip the verification workflow
5. **Iterate**: Use copilot-ask → copilot-exec → copilot-review cycle

## Interactive Workflow

Since Copilot CLI is interactive:

1. **Launch**: `copilot` in project directory
2. **Describe**: Explain what you want to build/change
3. **Preview**: Review Copilot's proposed changes
4. **Approve**: Explicitly approve each change
5. **Verify**: Run tests and checks
6. **Iterate**: Request adjustments if needed

## Limitations

- Interactive mode only (no direct scripting)
- Cannot run code or execute tests itself
- May not understand complex business logic
- Limited by context window size
- Requires manual verification
- Cannot guarantee correctness
- Uses quota per prompt

## Integration with Claude Code

When using this skill from Claude Code:

1. Verify git status is clean
2. Guide users to open a terminal
3. Navigate to the project directory
4. Launch `copilot`
5. Provide the formatted task description
6. Users interact with Copilot to approve changes
7. Verify changes after completion
8. Relay results back to the conversation

---

**Remember**: This skill MODIFIES code. Always review changes and run verification before committing!
