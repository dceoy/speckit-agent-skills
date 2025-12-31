---
name: speckit-specify
description: Create or update the feature specification from a natural language feature description.
allowed-tools: Bash, Read, Write, Grep, Glob
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create a plan for the spec. I am building with...
  - label: Clarify Spec Requirements
    agent: speckit.clarify
    prompt: Clarify specification requirements
    send: true
---

# Spec-Kit Specify

Create structured feature specifications from natural language descriptions. First step in the spec-kit workflow.

## When to Use

- User describes a new feature to build
- Starting a new project or feature
- Need to document requirements
- Converting ideas into structured specifications

## Execution Workflow

The user's input after the skill invocation is the feature description. The workflow:

1. **Generate concise branch name** (2-4 words) from feature description
2. **Check for existing branches** across remote, local, and specs directories to find next available number
3. **Run setup script** `.specify/scripts/bash/create-new-feature.sh` with calculated number and short-name
4. **Load spec template** from `.specify/templates/spec-template.md`
5. **Fill specification** with:
   - Feature overview (what, why, who)
   - User stories with priorities (P1, P2, P3)
   - Acceptance criteria for each story
   - Functional, non-functional requirements, and constraints
   - Make informed guesses for unclear aspects (document in Assumptions)
   - Limit `[NEEDS CLARIFICATION]` markers to maximum 3 critical items
6. **Validate quality** using spec quality checklist
7. **Present clarification questions** (if markers remain) with suggested answers in table format
8. **Update spec** with user's answers
9. **Report completion** with branch name, spec file path, and readiness for next phase

## Key Points

- Focus on **WHAT** users need and **WHY**, not HOW to implement
- Written for business stakeholders, not developers
- No tech stack, APIs, or code structure in spec
- Make informed guesses using context and industry standards
- Document assumptions in Assumptions section
- Maximum 3 `[NEEDS CLARIFICATION]` markers - prioritize by impact: scope > security/privacy > UX > technical details
- Success criteria must be measurable and technology-agnostic
- Each user story needs clear, testable acceptance criteria

## Next Steps

After creating spec.md:

- **Clarify** ambiguities with `speckit-clarify` (optional)
- **Plan** implementation with `speckit-plan`
