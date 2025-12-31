---
name: gemini-exec
description: Execute development tasks using Google Gemini CLI for code generation, refactoring, and modifications. Use when the user needs to create code, add features, refactor, fix bugs, or generate tests. Requires Gemini CLI installed.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Gemini Exec Skill

Use Google Gemini CLI to execute development tasks including code generation, refactoring, feature implementation, bug fixes, and testing. Leverage multimodal capabilities to generate code from designs.

## When to Use

- User wants to create new components, functions, or utilities
- User needs to refactor existing code
- User wants to add new features or extend functionality
- User needs to fix bugs or errors
- User wants to generate tests
- User wants to migrate dependencies or update patterns
- User has design files (mockups, wireframes, PDFs) to implement

## Prerequisites

Verify Gemini CLI is available:

```bash
gemini --version  # Should display installed version
```

Ensure authentication is configured (OAuth or API key).

## Basic Usage

### Step 1: Understand the Task

Identify the task type:

- **Code generation**: Create new components, functions, utilities
- **Refactoring**: Improve structure without changing behavior
- **Feature addition**: Extend existing functionality
- **Bug fix**: Correct errors and edge cases
- **Testing**: Add unit/integration tests
- **Migration**: Update dependencies or patterns
- **Design implementation**: Generate code from mockups, diagrams, PDFs

### Step 2: Gather Context

Before executing:

```bash
# Check current state
git status
git diff

# Check project context
cat GEMINI.md  # if exists

# Understand existing code
cat relevant-files
grep -r "existing-pattern"
```

### Step 3: Execute with Gemini

**Basic execution:**

```bash
gemini -p "TASK DESCRIPTION

Follow these guidelines:
- Follow existing code patterns and conventions
- Add appropriate error handling
- Include necessary imports
- Maintain code quality and readability
- Use proper types (TypeScript/etc)
- Add comments only for complex logic

Project context:
- Language: [detected from codebase]
- Framework: [detected from codebase]"
```

**With directory context:**

```bash
gemini --include-directories src,lib -p "TASK DESCRIPTION

Analyze existing patterns in src/ and lib/ before implementing.
Follow the same conventions."
```

**Generate from design files:**

```bash
gemini --include-files mockup.png,spec.pdf -p "Implement the user profile component:
- Follow the exact layout from mockup.png
- Implement requirements from spec.pdf
- Use existing component patterns from src/components/
- Include TypeScript types
- Add responsive styles"
```

### Step 4: Verify Changes

After Gemini executes:

```bash
# Review all changes
git status
git diff

# Check syntax and types
npm run lint  # or equivalent
npm run typecheck  # or equivalent

# Run tests
npm test  # or equivalent
```

### Step 5: Report Results

Provide a clear summary of what was done, what was verified, and any next steps.

## Example Tasks

### Code Generation

**Create components:**

```bash
gemini --include-directories src/components -p "Create a UserProfile component in src/components/ with:
- Props: name (string), email (string), avatar (optional string)
- Display user info in a card layout
- Include TypeScript types
- Follow existing component patterns from src/components/
- Use CSS modules for styling"
```

**Generate from mockups:**

```bash
gemini --include-files wireframe.png -p "Generate a React component based on this wireframe:
- Component structure matching the layout
- Proper props and state management
- CSS modules for styling
- Accessibility attributes
- TypeScript types"
```

**Generate from PDFs:**

```bash
gemini --include-files api-spec.pdf -p "Generate TypeScript types and API client from the API specification:
- Create types for all request/response schemas
- Generate API client with proper error handling
- Include authentication headers
- Add request/response interceptors"
```

**Create utilities:**

```bash
gemini -p "Create date formatting utilities in src/utils/date.ts:
- formatISO(date): ISO 8601 format
- formatRelative(date): 'X days ago' format
- formatLocale(date, locale): locale-specific format
- Include TypeScript types and JSDoc"
```

### Refactoring

**Extract functions:**

```bash
gemini -p "In src/components/LoginForm.tsx, extract validation logic into a separate validateCredentials function in src/utils/validation.ts. Maintain all existing functionality."
```

**Convert patterns:**

```bash
gemini --include-directories src/services -p "Refactor all promise chains in src/services/api.ts to use async/await. Add proper try-catch error handling."
```

**Improve structure:**

```bash
gemini -p "Split UserService in src/services/user.ts into:
- AuthService: login, logout, resetPassword
- ProfileService: getProfile, updateProfile
Maintain all functionality and update imports."
```

### Feature Addition

**Add validation:**

```bash
gemini -p "Add input validation to registration form in src/components/RegisterForm.tsx:
- Email: valid format, required
- Password: min 8 chars, uppercase, lowercase, number, special char
- Name: required, min 2 chars
- Display error messages below fields
- Disable submit until valid"
```

**Implement from requirements:**

```bash
gemini --include-files requirements.pdf -p "Implement the caching layer described in requirements.pdf:
- Use the architecture specified
- Follow existing service patterns
- Include proper error handling
- Add comprehensive tests"
```

### Bug Fixes

**Fix specific issues:**

```bash
gemini -p "Fix memory leak in src/hooks/useWebSocket.ts caused by not cleaning up WebSocket connection. Ensure cleanup in useEffect cleanup function."
```

**Address edge cases:**

```bash
gemini -p "Fix race condition in src/services/auth.ts where concurrent logins create duplicate sessions. Add proper locking or queueing."
```

### Testing

**Generate tests:**

```bash
gemini -p "Create comprehensive unit tests for src/utils/validation.ts:
- Test valid inputs
- Test invalid inputs
- Test edge cases
- Test error handling
- Use Jest
- Aim for 100% coverage"
```

**Integration tests:**

```bash
gemini -p "Create integration tests for authentication flow:
- Test successful login
- Test failed login
- Test password reset
- Test session management
- Mock external dependencies"
```

## Gemini-Specific Features

### Multimodal Code Generation

**From wireframes:**

```bash
gemini --include-files wireframe-1.png,wireframe-2.png,wireframe-3.png -p "Implement the complete user onboarding flow:
- Step 1 from wireframe-1.png
- Step 2 from wireframe-2.png
- Step 3 from wireframe-3.png
- Navigation between steps
- Form validation and state management"
```

**From architecture diagrams:**

```bash
gemini --include-files architecture.png -p "Implement the service layer shown in this diagram:
- Create all services and interfaces
- Implement dependency injection as shown
- Add proper error handling
- Include logging hooks at integration points"
```

**From design systems:**

```bash
gemini --include-files design-system.pdf,mockup.png -p "Create components matching the mockup:
- Follow design-system.pdf for colors, typography, spacing
- Implement mockup.png layout
- Use component library patterns
- Include responsive breakpoints"
```

### Google Search Grounding

```bash
gemini -p "Implement OAuth2 authentication using current best practices. Use Google Search to find latest security recommendations and implement them."
```

### Conversation Checkpointing

Multi-phase implementation (context preserved):

```bash
# Phase 1
gemini -p "First, create the base component structure for the dashboard"

# Phase 2 (context preserved)
gemini -p "Now add data fetching logic to the dashboard we just created"

# Phase 3
gemini -p "Finally, add error handling and loading states"
```

### Large Context Window

```bash
gemini --include-directories src,lib,tests,config -p "Refactor the entire authentication system:
- Analyze all auth-related code
- Modernize patterns
- Improve error handling
- Update tests"
```

## Best Practices

✅ **DO:**

- Review changes with `git diff` before declaring success
- Run tests after modifications
- Verify linting and type checking pass
- Follow existing code patterns
- Include proper error handling
- Use `--include-directories` to provide context
- Use `--include-files` for design references
- Leverage Google Search for current best practices
- Use conversation checkpointing for complex tasks

❌ **DON'T:**

- Skip verification steps
- Hardcode secrets or sensitive data
- Over-engineer solutions
- Ignore existing patterns
- Make changes without testing
- Skip error handling

## Verification Checklist

Before declaring success:

- [ ] Changes match what was requested
- [ ] No unexpected modifications
- [ ] Linting passes
- [ ] Type checking passes
- [ ] Tests pass
- [ ] Manual testing confirms behavior
- [ ] No security vulnerabilities
- [ ] No hardcoded secrets

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

**If execution fails:**

1. Check error message
2. Simplify the task
3. Provide more context
4. Try narrower scope with `--include-directories`

**If changes are incorrect:**

```bash
# Revert changes
git restore .

# Re-execute with better prompt
gemini -p "More specific task description..."
```

**If context window exceeded:**

```bash
# Narrow scope
gemini --include-directories src/specific-module -p "Task"

# Break into phases
gemini -p "First, implement core functionality"
gemini -p "Now add additional features"
```

## Output Format

After execution, report:

```markdown
## Task Completed: [Brief Description]

### Changes Made

- Created: `src/components/NewComponent.tsx`
- Modified: `src/services/api.ts`
- Deleted: `src/old/deprecated.ts`

### Summary

[What was done and why]

### Verification

- [✓] Lint passed
- [✓] Type check passed
- [✓] Tests passed (15 passing)
- [✓] Manual testing confirmed

### Next Steps

[Recommended follow-up actions, if any]
```

## Related Skills

- **gemini-ask**: For understanding existing code before making changes
- **gemini-review**: For reviewing changes after implementation
- **gemini-search**: For researching current best practices before implementing

## Tips for Better Results

1. **Be specific**: Include exact requirements and constraints
2. **Provide context**: Use `--include-directories` for existing patterns
3. **Use designs**: Attach mockups/PDFs with `--include-files`
4. **Break down**: Split complex tasks into phases
5. **Verify**: Always check changes before committing
6. **Search**: Use Google Search grounding for current best practices

## Advanced Usage

### Iterative Development

```bash
# Step 1: Core functionality
gemini -p "Implement basic user authentication"

# Step 2: Enhance (context preserved)
gemini -p "Add password reset to the authentication we just implemented"

# Step 3: Test
gemini -p "Create tests for the authentication system"

# Step 4: Review
# Use gemini-review skill
```

### Multi-Artifact Generation

```bash
gemini --include-files page-1.png,page-2.png,page-3.png -p "Create complete multi-page form:
- Page 1 from page-1.png (personal info)
- Page 2 from page-2.png (address)
- Page 3 from page-3.png (confirmation)
- State management across pages
- Validation and error handling"
```

### Integration from Specs

```bash
gemini --include-files openapi-spec.yaml -p "Generate complete API client from OpenAPI spec:
- TypeScript types for all schemas
- API methods for all endpoints
- Error handling and retries
- Authentication integration
- Request/response interceptors"
```

## Model Selection

Use `-m` flag for specific models:

```bash
# Faster model for simple tasks
gemini -m gemini-2.5-flash -p "Simple task"

# Pro model for complex tasks (default)
gemini -m gemini-2.5-pro -p "Complex task"
```

## Limitations

- Requires manual verification of changes
- May need multiple iterations for complex tasks
- Quality depends on prompt clarity and context provided
- Large refactorings may need breaking into phases

## Unique Capabilities vs Other Tools

**vs codex-exec:**

- ✅ Native multimodal support (generate from designs)
- ✅ Built-in Google Search grounding
- ✅ 1M token context window
- ✅ Conversation checkpointing
- ✅ Better for design-to-code workflows

**vs copilot-exec:**

- ✅ Multimodal code generation
- ✅ Google Search integration
- ✅ Larger context window
- ✅ Better for complex, multi-file changes

---

**Remember**: Always review changes with `git diff`, run tests, and verify code quality. Leverage Gemini's multimodal and search capabilities for better results.
