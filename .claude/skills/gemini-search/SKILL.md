---
name: gemini-search
description: Search the web using Google Gemini CLI with built-in Google Search grounding to find current documentation, best practices, solutions, and technical information. Use when the user needs to research libraries, find API documentation, troubleshoot errors, or learn about new technologies. Requires Gemini CLI installed.
allowed-tools: Bash, Read, Grep, Glob, WebFetch, WebSearch
---

# Gemini Search Skill

Use Google Gemini CLI with **built-in Google Search grounding** to research current information, documentation, and solutions. This is a **read-only research** skill with multimodal capabilities.

## When to Use

- User asks "what's the best library for X?"
- User needs current documentation or API references
- User wants to compare different libraries or approaches
- User has an error message and needs solutions
- User asks about best practices or security recommendations
- User wants to learn about a new technology or framework
- User needs to find tutorials or getting started guides
- User wants current standards (OWASP, WCAG, RFC)
- User has PDF/image documentation to analyze alongside web research

## Prerequisites

Verify Gemini CLI is available:

```bash
gemini --version  # Should display installed version
```

Ensure authentication is configured and internet connectivity.

## Basic Usage

### Step 1: Parse the Research Need

Extract what the user wants to know:

- Specific technology or library?
- What aspect (documentation, comparison, best practices)?
- Any version requirements?
- Context from current project?
- Any visual references (PDFs, diagrams)?

### Step 2: Gather Local Context (Optional)

Check current project context if relevant:

```bash
# Check current dependencies
cat package.json  # or requirements.txt, go.mod, etc.

# Check project context
cat GEMINI.md  # if exists

# Check versions
npm list --depth=0
```

### Step 3: Execute Gemini Search

**Gemini has built-in Google Search grounding** - just ask:

```bash
gemini -p "Research and provide comprehensive information about: [QUERY]

Include:
1. Direct answer to the question
2. Official documentation links
3. Best practices and recommended approaches
4. Code examples with explanations
5. Common pitfalls and how to avoid them
6. Alternative approaches if applicable
7. Source URLs for all information

Search context:
- Current year: 2026
- Looking for latest/current information
- Prefer official documentation over third-party sources
- Include security considerations if relevant

Use Google Search to find the most current and authoritative sources."
```

**With multimodal research:**

```bash
gemini --include-files api-spec.pdf -p "Extract information from this API specification and use Google Search to:
- Find official documentation for similar APIs
- Compare with current best practices
- Identify any outdated patterns
- Suggest modern alternatives"
```

### Step 4: Present Results

Format findings with proper citations:

````markdown
## Research: [Topic]

### Summary

[Direct answer with key findings in 2-3 sentences]

### Official Documentation

- [Library Name - Topic](https://url) - Official reference
- [API Docs](https://url) - API documentation

### Key Findings

**Best Practices:**

1. [Practice] - [Explanation] ([Source URL from Google Search])
2. [Practice] - [Explanation] ([Source URL])

**Common Pitfalls:**

- [Issue] - [How to avoid] ([Source URL])

### Code Examples

```language
// Source: [URL]
// Example: [Description]
[Working code example]
```

### Alternative Approaches

| Approach   | Pros | Cons | Use When | Source |
| ---------- | ---- | ---- | -------- | ------ |
| [Option 1] | ...  | ...  | ...      | [URL]  |
| [Option 2] | ...  | ...  | ...      | [URL]  |

### Sources

All information from Google Search via Gemini:

1. [Title](URL) - [Description]
2. [Title](URL) - [Description]
3. [Title](URL) - [Description]

Last verified: [Current date]
````

## Example Queries

### Documentation Lookup

```bash
gemini -p "Find official documentation for React Server Components in 2026:
- Installation and setup
- Core concepts and API
- Common use cases and examples
- Configuration options
- Migration from Pages Router

Use Google Search to find current official documentation."
```

### Library Comparison

```bash
gemini -p "Compare Prisma vs Drizzle vs Kysely for TypeScript database ORMs in 2026:
- Feature comparison
- Performance benchmarks
- Community adoption
- Learning curve
- Type safety
- Use case recommendations

Use Google Search to find recent comparisons and official documentation."
```

### Error Resolution

```bash
gemini -p "Research solutions for error: 'ECONNREFUSED' in Node.js:
- Root causes
- Common solutions
- Prevention strategies
- Related issues

Use Google Search to find Stack Overflow discussions and official Node.js docs."
```

### Best Practices

```bash
gemini -p "Find current security best practices for JWT authentication in 2026:
- Latest OWASP recommendations
- Token storage best practices
- Refresh token patterns
- Common vulnerabilities
- Security tools

Use Google Search to find official security standards and OWASP Top 10 2026."
```

### Getting Started

```bash
gemini -p "Find getting started guide for Bun runtime in 2026:
- Installation and setup
- Project structure
- Core concepts
- First app tutorial
- Best practices for beginners
- Migration from Node.js

Use Google Search to find official Bun documentation."
```

### Standards and Compliance

```bash
gemini -p "Find current WCAG 2.2 accessibility requirements for web applications:
- Latest specification details
- Implementation checklist
- Testing tools and techniques
- Common compliance issues
- Official compliance resources

Use Google Search to find W3C official documentation."
```

### Multimodal Research

```bash
gemini --include-files architecture-diagram.png -p "Analyze this architecture diagram and use Google Search to:
- Find official documentation for the patterns shown
- Compare with current best practices for this architecture
- Find similar reference architectures
- Identify any outdated patterns
- Suggest modern alternatives"
```

```bash
gemini --include-files api-spec.pdf -p "Extract API information from this PDF and use Google Search to:
- Find current best practices for this API pattern
- Compare with modern API design standards
- Find security recommendations
- Identify any deprecated patterns"
```

## Gemini-Specific Features

### Built-in Google Search Grounding

**Automatic web search integration:**

- No special flags needed - just ask
- Results include URLs automatically
- Real-time information access
- Cross-referenced from multiple sources

```bash
gemini -p "What's the latest version of Next.js in 2026 and what are the new features? Use Google Search."
```

### Multimodal Research

**Analyze documentation alongside web search:**

```bash
gemini --include-files internal-docs.pdf,diagram.png -p "Compare our internal documentation with current industry best practices:
- Use Google Search to find current standards
- Compare our approach with industry patterns
- Identify gaps or outdated practices
- Suggest modern improvements"
```

### Large Context Window

**Comprehensive research:**

```bash
gemini --include-directories docs --include-files spec.pdf -p "Analyze our complete documentation and use Google Search to:
- Find current best practices for our tech stack
- Compare our patterns with industry standards
- Identify outdated approaches
- Suggest modernization strategies"
```

### Conversation Checkpointing

**Multi-phase research:**

```bash
# Phase 1: Overview
gemini -p "Use Google Search: What is the current state of React state management in 2026?"

# Phase 2: Deep dive (context preserved)
gemini -p "Now search for: Detailed comparison of Zustand vs Jotai vs Valtio"

# Phase 3: Implementation (context preserved)
gemini -p "Find: Best practices for implementing Zustand in large applications"
```

## Best Practices

✅ **DO:**

- Always request Google Search explicitly in prompts
- Include URLs for all sources
- Prefer official documentation
- Verify information is current (2025-2026)
- Cross-reference multiple sources
- Note version-specific information
- Include security considerations
- Use multimodal for PDF/image documentation
- Cite all sources with complete URLs

❌ **DON'T:**

- Make or suggest code changes (use gemini-exec)
- Trust outdated information (pre-2024)
- Rely on single sources
- Present unverified information
- Skip source attribution
- Ignore official documentation

## Verification

After getting search results:

1. Verify URLs are accessible and relevant
2. Check dates on sources (prefer recent)
3. Confirm information matches official docs
4. Cross-reference conflicting information
5. Add context specific to user's project if applicable
6. Validate multimodal analysis accuracy

## Source Evaluation

**Prioritize (trust highly):**

- Official documentation and guides
- Official GitHub repositories
- CVE databases and security advisories
- MDN (for web standards)
- OWASP (for security)
- W3C (for web standards)
- RFC specifications
- Major framework official blogs

**Verify (use with validation):**

- Stack Overflow (check dates, votes, accepted answers)
- Reputable tech blogs (Google Developers, Microsoft Docs)
- GitHub issues and discussions
- Conference talks
- Developer advocates from official teams

**Use with caution:**

- Personal blogs (unless from known experts)
- Old tutorials (pre-2024 for rapidly changing tech)
- Unverified forum posts
- Content without proper sources

## Error Handling

**If Gemini not found:**

```
Gemini CLI is not available. Ensure it's installed and in your PATH.

Install options:
- npm install -g @google/gemini-cli
- brew install gemini-cli
```

**If authentication fails:**

```
Gemini CLI needs authentication. Run:
gemini

Then sign in with Google account or configure API key from aistudio.google.com/apikey
```

**If search returns no results:**

- Simplify the query
- Try alternative keywords
- Search for related concepts
- Use broader terms

**If results are outdated:**

- Explicitly request "2026" or "latest" in query
- Use WebFetch to verify official docs
- Ask for recent comparisons

**If results conflict:**

- Search for official stance
- Check version-specific differences
- Verify dates and determine which is current

**If context window exceeded:**

```bash
# Break into focused searches
gemini -p "First, search for: [PART_1]"
gemini -p "Now search for: [PART_2]"  # context preserved
```

## Integration with Codebase

After gathering information:

1. **Assess applicability** to current project
2. **Check compatibility** with existing dependencies
3. **Plan implementation** steps if adopting findings
4. **Suggest testing** approach

Example:

```bash
# After researching a library, check compatibility
npm info [package]@latest peerDependencies
npm install [package]@latest --dry-run
```

## Related Skills

- **gemini-ask**: For understanding existing code in your codebase
- **gemini-exec**: For implementing solutions found through research
- **gemini-review**: For validating code quality after implementation

## Tips for Better Results

1. **Be specific**: "What's the best React state management for large TypeScript apps?" vs "What's good for state?"
2. **Include context**: "For a Next.js 15 app with TypeScript..."
3. **Request sources**: "Include official documentation and recent comparisons"
4. **Specify timeframe**: "Current best practices in 2026"
5. **Define scope**: "Focus on security best practices"
6. **Use multimodal**: Attach PDFs/images with `--include-files`
7. **Explicit search**: "Use Google Search to find..."

## Use Cases

### Technology Evaluation

```bash
gemini -p "Use Google Search to evaluate GraphQL vs REST vs tRPC for TypeScript API in 2026:
- Current adoption trends
- Performance comparisons
- Developer experience
- Ecosystem maturity
- Use case recommendations"
```

### Migration Planning

```bash
gemini -p "Use Google Search to find migration guide from Webpack to Vite in 2026:
- Breaking changes
- Migration steps
- Common issues and solutions
- Performance improvements expected
- Plugin ecosystem comparison"
```

### Security Research

```bash
gemini -p "Use Google Search to find security vulnerabilities in React@18.3:
- Known CVEs
- Security advisories
- Recommended patches
- Best practices
- Security tools"
```

### Learning

```bash
gemini -p "Use Google Search to find beginner-friendly Rust tutorial for 2026:
- Official learning resources
- Interactive tutorials
- Project-based learning
- Best practices for newcomers
- Community recommendations"
```

### Troubleshooting

```bash
gemini -p "Use Google Search for solutions to 'Module not found' error in Vite:
- Common causes
- Solutions from Stack Overflow
- Official issue tracker discussions
- Prevention strategies"
```

### Multimodal + Search

```bash
gemini --include-files error-screenshot.png -p "Analyze this error screenshot and use Google Search to:
- Identify the error
- Find common causes
- Search for solutions
- Find official documentation about this error
- Provide step-by-step fix"
```

## Advanced Usage

### Multi-Query Research

```bash
# Query 1: Current state
gemini -p "Use Google Search: What is the current recommended approach for SSR in React 2026?"

# Query 2: Specific implementation (context preserved)
gemini -p "Now search for: Implementation examples for React Server Components"

# Query 3: Gotchas (context preserved)
gemini -p "Search for: Common problems with React Server Components and solutions"
```

### Version-Specific Research

```bash
gemini -p "Use Google Search to find information specifically for TypeScript 5.6:
- New features in this version
- Breaking changes from 5.5
- Migration guide
- Official release notes
Ensure results are version-specific."
```

### Comparison Research

```bash
gemini -p "Use Google Search for detailed comparison of Astro vs Next.js vs Remix in 2026:
- Build performance benchmarks
- Developer experience
- Ecosystem and plugin support
- Production optimization
- SEO capabilities
- Use case recommendations
Include recent benchmarks and official documentation."
```

### Standards Research

```bash
gemini -p "Use Google Search to find current OWASP Top 10 2026:
- Complete list with descriptions
- Implementation guidelines for each
- Testing methodologies
- Common vulnerabilities examples
- Prevention strategies
- Official OWASP resources"
```

### Multimodal Standards Research

```bash
gemini --include-files current-implementation.pdf -p "Analyze our current security implementation from this PDF and use Google Search to:
- Compare with current OWASP Top 10 2026
- Find gaps in our implementation
- Search for best practices we're missing
- Find modern security patterns
- Identify deprecated approaches"
```

## Limitations

- Requires internet connectivity
- Search quality depends on available online information
- Cannot execute or test code
- Cannot make code modifications
- Limited to publicly available information
- Multimodal analysis quality depends on input quality

## Unique Capabilities vs Other Tools

**vs codex-search:**

- ✅ **Native Google Search integration** (built-in, no flags needed)
- ✅ Multimodal research (PDFs, images, diagrams)
- ✅ 1M token context window
- ✅ Conversation checkpointing
- ✅ Superior for visual documentation analysis

**vs copilot-search:**

- ✅ Built-in Google Search grounding
- ✅ Multimodal capabilities
- ✅ Larger context window
- ✅ Better for comprehensive research

**Key Advantage: Gemini's Google Search grounding is built-in and automatic** - no special configuration, flags, or external tools needed.

---

**Remember**: This skill is READ-ONLY research. Always cite sources with complete URLs from Google Search. Leverage Gemini's built-in search and multimodal capabilities for comprehensive, current information.
