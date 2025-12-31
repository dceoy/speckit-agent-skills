# Codex CLI Agents for Claude Code

This directory contains specialized Claude Code agents that integrate OpenAI Codex CLI capabilities into your development workflow.

## What Are These Agents?

These are **autonomous Claude Code agents** that can be launched using the Task tool. Each agent specializes in a specific development task and uses OpenAI Codex CLI to accomplish its mission.

## Available Agents

### 1. codex-ask

**Purpose:** Answer questions about code (read-only)

**Launch:**

```
I'll launch the codex-ask agent to find out how authentication works
```

**What it does:**

- Uses Codex to analyze code
- Provides detailed answers
- Includes file references and line numbers
- Shows code examples
- NEVER modifies code

**When to use:**

- Understanding existing code
- Exploring architecture
- Finding implementations
- Learning patterns
- Debugging assistance

**Example queries:**

- "How does authentication work?"
- "Where is email validation implemented?"
- "What's the database schema?"
- "Why might the login be failing?"

---

### 2. codex-exec

**Purpose:** Execute development tasks with code modifications

**Launch:**

```
I'll launch the codex-exec agent to add email validation to the form
```

**What it does:**

- Generates new code
- Refactors existing code
- Adds features
- Fixes bugs
- Creates tests
- MODIFIES code files

**When to use:**

- Code generation
- Refactoring
- Feature implementation
- Bug fixes
- Test creation

**Example tasks:**

- "Add email validation to ContactForm"
- "Refactor UserService for better separation"
- "Fix the memory leak in useWebSocket hook"
- "Create tests for authentication flow"

---

### 3. codex-review

**Purpose:** Perform comprehensive code reviews

**Launch:**

```
I'll launch the codex-review agent to review the uncommitted changes
```

**What it does:**

- Identifies bugs and issues
- Finds security vulnerabilities
- Detects performance problems
- Suggests improvements
- Provides actionable feedback
- NEVER modifies code

**When to use:**

- Pre-commit checks
- Pull request reviews
- Security audits
- Performance analysis
- Code quality assessment

**Review types:**

- General comprehensive review
- Security-focused review
- Performance-focused review
- Architecture review
- Accessibility review

---

## How to Use These Agents

Simply describe your task to Claude Code, and it will launch the appropriate agent:

```
User: "Can you help me understand how authentication works?"
→ Launches codex-ask agent

User: "Add input validation to the registration form"
→ Launches codex-exec agent

User: "Review my changes for security issues"
→ Launches codex-review agent
```

You can also explicitly request an agent by name if desired.

## Workflow Patterns

### Pattern 1: Understand → Execute → Review

```
1. User: "I need to improve error handling in the payment service"
2. Launch codex-ask: "How is error handling currently implemented?"
3. Launch codex-exec: "Improve error handling based on findings"
4. Launch codex-review: "Review the error handling changes"
```

### Pattern 2: Pre-Commit Workflow

```
1. Make changes to code
2. Launch codex-review: "Review uncommitted changes"
3. Launch codex-exec: "Fix issues identified in review"
4. Launch codex-review: "Final verification"
5. Commit with confidence
```

### Pattern 3: Feature Development

```
1. Launch codex-ask: "What validation patterns are used?"
2. Launch codex-exec: "Add email validation following existing patterns"
3. Launch codex-exec: "Create tests for the new validation"
4. Launch codex-review: "Review the implementation"
```

### Pattern 4: Troubleshooting

```
1. Launch codex-ask: "What could cause this error in UserProfile?"
2. Launch codex-review: "Check UserProfile for potential bugs"
3. Launch codex-exec: "Fix the identified issues"
4. Test manually
```

## Agent Coordination

These agents can work together seamlessly:

| First Agent  | Then Agent   | Purpose                      |
| ------------ | ------------ | ---------------------------- |
| codex-ask    | codex-exec   | Understand, then modify      |
| codex-exec   | codex-review | Implement, then verify       |
| codex-review | codex-exec   | Identify issues, then fix    |
| codex-ask    | codex-review | Explore, then assess quality |

## Prerequisites

Before using these agents:

1. **Codex CLI** (assumes already installed)
   - Verify: `codex --version`

2. **Authentication configured**
   - ChatGPT Plus/Pro/Team/Enterprise subscription
   - Or OpenAI API key in `~/.codex/config.toml`

3. **Internet connection**
   - All Codex operations require API access

## Configuration

### Global Config

`~/.codex/config.toml`:

```toml
[general]
model = "gpt-4o"

[execution]
auto_approve = false

[api]
# If using API key:
# key = "sk-..."
```

### Project Config

`.codex/config.toml` (optional):

```toml
[project]
name = "cloudflare-pages-react-form"
language = "typescript"
```

## Agent Behavior

### What Agents Can Do

- ✅ Run Bash commands (codex CLI, git, project tools)
- ✅ Read files to understand context
- ✅ Search code with Grep
- ✅ Find files with Glob
- ✅ Create/modify files (codex-exec only)
- ✅ Report results with formatting

### What Agents Won't Do

- ❌ Make changes without explanation
- ❌ Skip verification steps
- ❌ Ignore errors
- ❌ Commit code automatically
- ❌ Deploy to production

## Safety Features

### codex-ask Agent

- **Read-only** - Never modifies code
- Verifies file references
- Provides evidence-based answers

### codex-exec Agent

- Reviews changes before declaring success
- Runs linter and type checker
- Executes tests
- Prompts for approval by default
- Shows git diff

### codex-review Agent

- **Read-only** - Never modifies code
- Categorizes issues by severity
- Provides fix suggestions
- Includes positive observations

## Tips for Best Results

### 1. Be Specific

✅ Good: "Review src/auth.ts for security vulnerabilities"
❌ Poor: "Review the code"

### 2. Provide Context

✅ Good: "Add email validation to ContactForm.tsx using regex pattern"
❌ Poor: "Add validation"

### 3. One Task Per Agent

✅ Good: Launch codex-exec for one feature at a time
❌ Poor: "Add validation, refactor, and create tests" (too much)

### 4. Use Appropriate Agent

- Questions? → codex-ask
- Changes? → codex-exec
- Review? → codex-review

### 5. Iterate

Use ask → exec → review cycle for complex tasks

## Troubleshooting

### "Codex CLI not found"

**Solution:** Ensure Codex CLI is installed and available in your PATH

```bash
which codex  # Verify installation
codex --version  # Check version
```

### "Authentication failed"

**Solution:**

1. Run `codex` and follow authentication flow
2. Check ChatGPT subscription at https://chatgpt.com
3. Or configure API key in `~/.codex/config.toml`

### "Agent not responding"

**Solution:**

- Check internet connection
- Verify Codex CLI is working: `codex --version`
- Check for API rate limits

### "Unexpected results"

**Solution:**

- Provide more specific instructions
- Break task into smaller steps
- Review agent's output for errors

## Examples

### Example 1: Understanding Code

```
User: How does the form validation work in this app?
Agent: I'll launch the codex-ask agent to analyze form validation.

[codex-ask agent runs]
→ Provides detailed explanation with file references
→ Shows code examples from actual implementation
→ Explains validation patterns used
```

### Example 2: Adding a Feature

```
User: Add a dark mode toggle to the app

Agent: I'll launch the codex-exec agent to implement dark mode.

[codex-exec agent runs]
→ Creates dark mode context
→ Adds toggle component
→ Updates styles
→ Tests the implementation
→ Reports: "Dark mode toggle added successfully"
```

### Example 3: Code Review

```
User: Review my changes before I commit

Agent: I'll launch the codex-review agent to review uncommitted changes.

[codex-review agent runs]
→ Analyzes all changed files
→ Identifies 2 important issues
→ Suggests 3 improvements
→ Notes 2 good practices
→ Provides fix recommendations
```

### Example 4: Complete Workflow

```
User: I need to improve the authentication system

Agent: I'll use multiple agents to handle this comprehensively.

Step 1: Understanding
[Launches codex-ask: "How is authentication currently implemented?"]
→ Gets complete auth flow analysis

Step 2: Review
[Launches codex-review: "Security review of authentication code"]
→ Identifies 2 security issues
→ Suggests improvements

Step 3: Implementation
[Launches codex-exec: "Fix security issues and add rate limiting"]
→ Implements fixes
→ Adds rate limiting
→ Updates tests

Step 4: Verification
[Launches codex-review: "Final security review of auth changes"]
→ Confirms issues are fixed
→ No new issues found
→ Ready to commit
```

## Agent Files

```
.claude/agents/
├── README.md        # This file
├── codex-ask.md     # Question-answering agent
├── codex-exec.md    # Execution agent
└── codex-review.md  # Code review agent
```

## Related Resources

- `.codex/skills/codex-cli/` - Codex CLI subagents (for use within Codex, not Claude)
- https://developers.openai.com/codex/cli/ - Official Codex CLI documentation
- https://github.com/openai/codex - Codex CLI GitHub repository

## Architecture

```
┌─────────────────────────────────────┐
│         Claude Code                  │
│  (Main AI Agent Interface)           │
└────────────┬────────────────────────┘
             │
             │ Launches
             ▼
┌─────────────────────────────────────┐
│   Specialized Codex Agents           │
│  ┌──────────┬──────────┬──────────┐ │
│  │   ask    │   exec   │  review  │ │
│  └──────────┴──────────┴──────────┘ │
└────────────┬────────────────────────┘
             │
             │ Uses
             ▼
┌─────────────────────────────────────┐
│      OpenAI Codex CLI                │
│  (Local Terminal Tool)               │
└────────────┬────────────────────────┘
             │
             │ API Calls
             ▼
┌─────────────────────────────────────┐
│      OpenAI API                      │
│  (GPT-4o, o1-mini, etc.)             │
└─────────────────────────────────────┘
```

## Comparison: Agents vs Direct CLI

| Approach       | When to Use                 | Pros                             | Cons                     |
| -------------- | --------------------------- | -------------------------------- | ------------------------ |
| **Agents**     | Multi-step tasks via Claude | Autonomous, thorough, structured | More verbose             |
| **Direct CLI** | Manual terminal control     | Full control, fastest            | Requires Codex expertise |

**Recommendation:** Use agents when working within Claude Code for autonomous, multi-phase task execution.

## Adding New Agents

To create a new agent:

1. Create `.claude/agents/<agent-name>.md`
2. Add YAML frontmatter: `name`, `description`, `version`
3. Write detailed multi-phase execution plan
4. Include error handling and verification steps
5. Update this README with agent description

## Version History

- **v1.0.0** (2025-12-31) - Initial release
  - codex-ask: Question-answering agent (read-only)
  - codex-exec: Task execution agent (with modifications)
  - codex-review: Code review agent (read-only)

## Support

For issues:

- **Codex CLI:** https://github.com/openai/codex/issues
- **Claude Code:** https://github.com/anthropics/claude-code/issues
- **These agents:** File issue in this repository

---

**Ready to use Codex CLI agents! Start by asking Claude Code to launch the appropriate agent for your task.**
