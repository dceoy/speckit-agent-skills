---
name: copilot-ask
description: Ask GitHub Copilot CLI questions about code to understand implementations, architecture, patterns, and debugging. Use when the user asks how code works, where something is implemented, what patterns are used, or needs to understand existing code. Requires Copilot CLI installed.
allowed-tools: Bash, Read, Grep, Glob
---

# Copilot Ask Skill

Use GitHub Copilot CLI to answer questions about code without making modifications. This is a **read-only** analysis skill.

## When to Use

- User asks "how does X work?"
- User wants to find where something is implemented
- User needs to understand architecture or patterns
- User is debugging and needs to understand code flow
- User asks "what does this code do?"

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

### Step 1: Parse the Question

Extract what the user wants to know. Good questions include:

- File/component context
- Specific functionality
- Desired answer format

### Step 2: Launch Copilot CLI

Since Copilot CLI is interactive, guide the user to launch it:

```bash
# Navigate to the code directory
cd /path/to/project

# Launch Copilot CLI
copilot
```

### Step 3: Ask the Question

Provide a well-formatted prompt for Copilot CLI:

```
Explain how [FEATURE/COMPONENT] works in this codebase.

Please provide:
1. Direct answer to the question
2. Specific file paths and line numbers
3. Code examples from the actual codebase
4. Related concepts or dependencies

Do NOT make any changes - this is read-only analysis.
```

### Step 4: Present Answer

Format the response with:

- **Summary**: 1-2 sentence direct answer
- **Details**: In-depth explanation
- **File References**: Specific paths with line numbers
- **Code Examples**: Relevant snippets
- **Related Info**: Dependencies, gotchas, context

## Example Queries

### Understanding Flow

```
Explain how user authentication works in this app. Include all files involved, the complete flow, and security measures. Do NOT modify code.
```

### Finding Implementation

```
Where is email validation implemented? Show all locations with file paths and line numbers. Do NOT modify code.
```

### Architecture Questions

```
What's the overall architecture of this application? Describe patterns used, component organization, and data flow. Do NOT modify code.
```

### Debugging

```
What could cause 'Cannot read property of undefined' in UserProfile component? Analyze potential causes with specific line references. Do NOT modify code.
```

## Output Format

Structure answers like this:

````markdown
## Answer: [Question]

### Summary

[Direct answer in 1-2 sentences]

### Details

[Comprehensive explanation]

### File References

- `src/auth/login.ts:45-67` - Login handler implementation
- `src/middleware/auth.ts:23` - Authentication middleware

### Code Examples

```typescript
// From src/auth/login.ts:45
export async function handleLogin(credentials: Credentials) {
  // ... code snippet ...
}
```

### Related Information

- Uses JWT for token management
- Session timeout is 24 hours
- See `src/config/auth.ts` for configuration
````

## Best Practices

✅ **DO:**

- Always include file paths with line numbers
- Show actual code from the codebase
- Explain "why" not just "what"
- Mention related files or concepts
- Verify Copilot returned accurate information
- Use the `/model` command to switch AI models if needed

❌ **DON'T:**

- Make or suggest code changes (use copilot-exec for that)
- Execute code or run tests
- Modify files
- Assume information without verification

## Verification

After getting Copilot's response:

1. Verify file paths exist and are correct
2. Check line numbers are accurate
3. Confirm code examples match current code
4. Add any missing context from your knowledge

## Model Selection

GitHub Copilot CLI defaults to Claude Sonnet 4.5 but supports other models:

```
/model
```

Then choose from:

- Claude Sonnet 4.5 (default, recommended)
- Claude Sonnet 4
- GPT-5

## Error Handling

**If Copilot not found:**

```
GitHub Copilot CLI is not available.

Ensure it's installed and available in your PATH:
- Check: which copilot
- Verify: copilot --version
```

**If authentication fails:**

```
Copilot CLI needs authentication. Run:
copilot

Then use the /login command and follow the prompts to sign in with GitHub.
```

**If answer is unclear:**

- Ask a more specific question
- Provide more context (file names, features)
- Break complex questions into smaller parts
- Try switching models with `/model`

## Interactive Workflow

Since Copilot CLI is interactive, the workflow is:

1. **Launch**: `copilot` in the project directory
2. **Ask**: Type your question in natural language
3. **Review**: Copilot analyzes and responds
4. **Iterate**: Ask follow-up questions as needed
5. **Exit**: Use Ctrl+C or exit command

## Related Skills

- **copilot-exec**: For making code changes
- **copilot-review**: For code quality assessment

## Tips for Better Results

1. **Be specific**: "How does JWT validation work in auth middleware?" vs "How does auth work?"
2. **Include context**: "In the user registration flow..."
3. **Specify scope**: "Focus on src/components/..."
4. **Request format**: "Explain with code examples and file paths"

## Quota Management

Each prompt uses one premium request from your monthly quota. Use wisely:

- Combine related questions into one prompt when possible
- Be specific to get targeted answers
- See GitHub docs for quota limits and usage

## Limitations

- Interactive mode only (no direct scripting interface)
- Cannot execute or test code
- Cannot make modifications
- Limited to static code analysis
- May not understand business logic context
- Uses quota per prompt

## Integration with Claude Code

When using this skill from Claude Code:

1. Guide users to open a terminal
2. Navigate to the project directory
3. Launch `copilot`
4. Provide the formatted question
5. Users interact with Copilot directly
6. Relay answers back to the conversation

---

**Remember**: This skill is READ-ONLY. For code modifications, use the `copilot-exec` skill.
