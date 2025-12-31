---
name: gemini-ask
description: Ask Google Gemini questions about code to understand implementations, architecture, patterns, and debugging. Use when the user asks how code works, where something is implemented, what patterns are used, or needs to understand existing code. Requires Gemini CLI installed.
allowed-tools: Bash, Read, Grep, Glob
---

# Gemini Ask Skill

Use Google Gemini CLI to answer questions about code without making modifications. This is a **read-only** analysis skill with multimodal capabilities.

## When to Use

- User asks "how does X work?"
- User wants to find where something is implemented
- User needs to understand architecture or patterns
- User is debugging and needs to understand code flow
- User asks "what does this code do?"
- User wants to analyze diagrams, screenshots, or PDFs
- User needs to understand visual documentation

## Prerequisites

Verify Gemini CLI is available:

```bash
gemini --version  # Should display installed version
```

Ensure authentication is configured (OAuth or API key).

## Basic Usage

### Step 1: Parse the Question

Extract what the user wants to know:

- File/component context
- Specific functionality
- Desired answer format
- Any visual references (images, PDFs)

### Step 2: Gather Context (Optional)

Check project context if relevant:

```bash
# Check project-specific context
cat GEMINI.md  # if exists

# Check what exists
ls -la

# For specific questions
grep -r "pattern" --include="*.ts"
```

### Step 3: Execute Gemini Query

Run Gemini in read-only mode:

**Text-only query:**

```bash
gemini -p "Answer this question about the codebase: [QUESTION]

Provide:
1. Direct answer to the question
2. Specific file paths and line numbers
3. Code examples from the actual codebase
4. Related concepts or dependencies

Context: I am analyzing the code to understand [SPECIFIC_ASPECT].
Do NOT make any changes - this is read-only analysis."
```

**With directory context:**

```bash
gemini --include-directories src,lib -p "Answer this question about the codebase: [QUESTION]

Include relevant code from src/ and lib/ directories.
Provide file paths and line numbers.
Do NOT make any changes."
```

**Multimodal query (with images, PDFs):**

```bash
gemini --include-files diagram.png -p "Analyze this architecture diagram and explain:
- What components are shown
- How they interact
- Which files in the codebase implement these components
- Any potential issues or improvements

Do NOT make any changes - this is read-only analysis."
```

### Step 4: Present Answer

Format the response with:

- **Summary**: 1-2 sentence direct answer
- **Details**: In-depth explanation
- **File References**: Specific paths with line numbers (e.g., `src/auth/login.ts:45-67`)
- **Code Examples**: Relevant snippets
- **Related Info**: Dependencies, gotchas, context
- **Sources**: If Google Search grounding was used

## Example Queries

### Understanding Flow

```bash
gemini -p "Explain how user authentication works in this app. Include all files involved, the complete flow, and security measures."
```

### Finding Implementation

```bash
gemini --include-directories src -p "Where is email validation implemented? Show all locations with file paths and line numbers."
```

### Architecture Questions

```bash
gemini -p "What's the overall architecture of this application? Describe patterns used, component organization, and data flow."
```

### Debugging

```bash
gemini -p "What could cause 'Cannot read property of undefined' in UserProfile component? Analyze potential causes with specific line references."
```

### Visual Analysis

```bash
gemini --include-files architecture-diagram.pdf -p "Analyze this architecture diagram:
- Identify all components and their responsibilities
- Explain data flow between components
- Compare with current implementation in src/
- Identify any discrepancies"
```

### Compare Design with Implementation

```bash
gemini --include-files mockup.png --include-directories src/components -p "Compare the mockup with the actual implementation in src/components/UserProfile:
- Visual fidelity
- Missing features
- Implementation differences
- Suggestions for alignment"
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

[Explanation of this code]

### Related Information

- Uses JWT for token management
- Session timeout is 24 hours
- See `src/config/auth.ts` for configuration

### Sources

[If Gemini used Google Search grounding, include URLs]
````

## Gemini-Specific Features

### Multimodal Understanding

Analyze visual documentation:

```bash
# Architecture diagrams
gemini --include-files arch.png -p "Explain the architecture shown in this diagram"

# PDF documentation
gemini --include-files spec.pdf -p "Summarize the API specification and show which parts are implemented"

# Screenshots
gemini --include-files screenshot.png -p "What error is shown in this screenshot and what might cause it?"

# Sketches/wireframes
gemini --include-files sketch.jpg -p "What components would be needed to implement this sketch?"
```

### Google Search Grounding

For current information:

```bash
gemini -p "How does this authentication implementation compare to current best practices? Use Google Search to find latest OWASP recommendations."
```

### Large Context Window

Analyze entire projects:

```bash
gemini --include-directories src,lib,tests -p "Analyze the complete architecture of this application with all source, library, and test code."
```

### Conversation Checkpointing

Multi-part analysis (context preserved automatically):

```bash
# First question
gemini -p "Explain the database layer architecture"

# Follow-up (context preserved)
gemini -p "Now explain how the API layer interacts with the database we just discussed"
```

## Best Practices

✅ **DO:**

- Always include file paths with line numbers
- Show actual code from the codebase
- Explain "why" not just "what"
- Mention related files or concepts
- Verify Gemini returned accurate information
- Use `--include-files` for visual references
- Use `--include-directories` to focus on specific code areas
- Leverage Google Search for current best practices

❌ **DON'T:**

- Make or suggest code changes (use gemini-exec for that)
- Execute code or run tests
- Modify files
- Assume information without verification
- Skip multimodal capabilities when relevant

## Verification

After getting Gemini's response:

1. Verify file paths exist and are correct
2. Check line numbers are accurate
3. Confirm code examples match current code
4. Add any missing context from your knowledge
5. Validate sources from Google Search if used

## Error Handling

**If Gemini not found:**

```
Gemini CLI is not available in PATH. Ensure it is installed per the prerequisites in README.md.
```

**If authentication fails:**

```
Gemini CLI needs authentication. Run:
gemini

Then follow the prompts to sign in with Google account or configure API key from aistudio.google.com/apikey
```

**If answer is unclear:**

- Ask a more specific question
- Provide more context (file names, features)
- Break complex questions into smaller parts
- Use `--include-directories` to narrow scope

**If context window exceeded:**

```bash
# Narrow the scope
gemini --include-directories src/auth -p "Question about authentication only"

# Or break into multiple questions
```

## Related Skills

- **gemini-exec**: For making code changes
- **gemini-review**: For code quality assessment
- **gemini-search**: For researching current information

## Tips for Better Results

1. **Be specific**: "How does JWT validation work in auth middleware?" vs "How does auth work?"
2. **Include context**: "In the user registration flow..."
3. **Specify scope**: "Focus on src/components/..." with `--include-directories`
4. **Request format**: "Explain with code examples and file paths"
5. **Use multimodal**: Attach diagrams/PDFs with `--include-files`
6. **Leverage search**: "Use Google Search to compare with current best practices"

## Advanced Usage

### Multi-File Analysis

```bash
gemini --include-files src/auth/*.ts,config/*.json -p "Analyze the complete authentication system including all auth files and configuration"
```

### Compare Multiple Artifacts

```bash
gemini --include-files design-v1.pdf,design-v2.pdf -p "Compare these two design versions and explain the differences"
```

### Context-Aware Analysis

```bash
gemini --include-directories src,docs -p "Explain the feature documented in docs/ and show its implementation in src/"
```

## Limitations

- Cannot execute or test code
- Cannot make modifications
- Read-only static analysis
- May not understand business logic context without additional context
- Multimodal analysis quality depends on image/PDF quality

## Unique Capabilities vs Other Tools

**vs codex-ask:**

- ✅ Native multimodal support (images, PDFs, sketches)
- ✅ Built-in Google Search grounding
- ✅ 1M token context window
- ✅ Conversation checkpointing

**vs copilot-ask:**

- ✅ Multimodal understanding
- ✅ Google Search integration
- ✅ Larger context window
- ✅ Better for visual documentation analysis

---

**Remember**: This skill is READ-ONLY. For code modifications, use the `gemini-exec` skill. Leverage Gemini's multimodal and search capabilities for comprehensive code understanding.
