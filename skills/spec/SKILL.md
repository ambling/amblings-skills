---
name: spec
description: Specification stage - create/update spec.md
---

# spec

You are now in the specification stage for the feature. Your task is to draft (or update if it exists) the `spec.md` file. Your specific instruction is: `'{{args}}'`.

## Purpose

The **spec.md** document defines **WHAT** to build - the functional requirements and goals of the feature. This is the first document in the documentation hierarchy and must be completed before any design work begins.

## Before You Begin

**First**, leverage `superpowers:brainstorming` to explore user intent, requirements, and initial design. The brainstorming output will be at:
`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

Use this as input to create a formal spec document. The brainstorming output contains the exploration and design thinking - your spec should formalize the WHAT (requirements) from it.

## Documentation Location

- **Top-level feature**: `docs/specs/YYYY-MM-DD_<feature_name>/spec.md`
- **Sub-feature/component**: `docs/specs/YYYY-MM-DD_<feature_name>/<component>/spec.md`

The date in the directory name (YYYY-MM-DD format) should be the current date when the spec is created.

## Content Requirements

Your spec.md must include:

### 1. Feature Overview
- Context and background
- Goals and objectives
- Problem statement

### 2. Functional Requirements
- **Core Capabilities**: What the system should do
- **User Stories**: User perspectives and use cases
- **Use Cases**: Detailed scenarios

### 3. Non-Functional Requirements
- **Performance**: Response times, throughput, scalability
- **Security**: Authentication, authorization, data protection
- **Reliability**: Availability, fault tolerance
- **Scalability**: Growth expectations

### 4. Deployment Requirements
- Short-term deployment approach
- Long-term deployment approach
- Infrastructure needs

### 5. Testing Requirements
- Test types needed (unit, integration, E2E)
- Acceptance criteria at high level

## Guidelines

**Focus on WHAT, not HOW:**
- ✅ "System shall support user authentication"
- ❌ "Use JWT tokens stored in Redis"

**Keep it high-level and concise:**
- Remove implementation details (file names, function signatures, code examples)
- Use diagrams for clarity when needed
- Define clear acceptance criteria

**Do NOT include:**
- Architecture diagrams
- Module breakdowns
- Interface definitions
- Code examples or pseudo-code
- File paths or function names

## Process

1. Read the instruction: `'{{args}}'`
2. Research the feature context if needed
3. Draft the spec.md following the content requirements
4. Review against guidelines to ensure it focuses on WHAT, not HOW
5. Request user to review and provide feedback

## Progress Tracking

After completing spec.md, update progress tracking:

### Update Notes File

Create/update `docs/coordinate/<feature>/spec_notes.md`:

```markdown
# Spec Notes: <Feature Name>

**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**Output**: `docs/specs/YYYY-MM-DD_<feature>/spec.md`
**Date**: [YYYY-MM-DD]
**Started**: [timestamp]
**Completed**: [timestamp]

## Summary
[Description of what was specified]

## Key Requirements
- [Requirement 1]
- [Requirement 2]

## Issues
[Any blockers or questions for user]

## User Feedback
[Record user approval and any comments]
```

### Output Status Report

After completion, provide:

```markdown
## Spec Phase Complete

**Feature**: <feature_name>
**Status**: COMPLETED
**Output**: `docs/specs/<feature>/YYYY-MM-DD-spec.md`

### Summary
- [Brief description of what was specified]

### Key Requirements
- [List 3-5 key functional requirements]

### Ready for Review
- Awaiting user approval for design phase
```

## Next Steps After Spec

After the spec.md is reviewed and approved, the next phase is **design.md** (HOW to design the system).

**You should NOT write any code in this stage.**
