---
name: design
description: Design stage for feature development - create/update design.md
---

# design

You are now in the design stage for the feature. Your task is to draft (or update if it exists) the `design.md` file. Your specific instruction is: `'{{args}}'`.

## Purpose

The **design.md** document defines **HOW** the system is designed at the architecture and module level. This document bridges the gap between the high-level spec.md and the detailed implementation_plan.md.

## Before You Begin

**First**, leverage `superpowers:brainstorming` which has already explored the design space. The brainstorming output is at:
`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

Use this as input to create a formal design document. The brainstorming output contains the initial design exploration - your design document should formalize the HOW (architecture, modules, interfaces, design decisions) from it.

## Documentation Location

- **Top-level feature**: `docs/specs/YYYY-MM-DD_<feature_name>/design.md`
- **Sub-feature/component**: `docs/specs/YYYY-MM-DD_<feature_name>/<component>/design.md`

The date in the directory name (YYYY-MM-DD format) should be the current date when the design is created.

## Content Requirements

Your design.md must include:

### 1. Architecture Design

#### 1.1 High-Level Architecture
- Architecture diagram showing major components
- Component boundaries and relationships
- Data flow overview

#### 1.2 Component Interaction Architecture
- How components communicate
- Request/response flows
- Event-driven interactions (if applicable)

### 2. Module Design

For each module, document:

#### Module X
**Scope:** What this module does
**Responsibilities:** Key responsibilities
**Interfaces:**
- **Input:** What the module receives
- **Output:** What the module produces
- **Dependencies:** What the module depends on

### 3. Key Design Decisions

For each significant design decision:

#### Decision X
**Decision:** What was decided
**Rationale:** Why this decision was made
**Trade-offs:** Alternatives considered

## Guidelines

**Focus on HOW the system is designed:**
- ✅ Architecture diagrams
- ✅ Module scopes and responsibilities
- ✅ Interface definitions (inputs/outputs)
- ✅ Design decisions with rationale

**Do NOT include:**
- ❌ Pseudo-code
- ❌ Implementation details
- ❌ File paths or function names
- ❌ Code examples
- ❌ Step-by-step implementation instructions

## Design Principles

Validate your design against SOLID principles:

1. **Single Responsibility Principle**: Each module has one reason to change
2. **Open/Closed Principle**: Open for extension, closed for modification
3. **Liskov Substitution Principle**: Subtypes substitutable for base types
4. **Interface Segregation Principle**: Clients depend only on needed interfaces
5. **Dependency Inversion Principle**: Depend on abstractions, not concretions

**In your design document, report any violations with justifications.**

## Process

1. Read the instruction: `'{{args}}'`
2. Review the spec.md to understand WHAT needs to be built
3. Design the system architecture and modules
4. Create architecture and interaction diagrams
5. Document module scopes, responsibilities, and interfaces
6. Record key design decisions with rationale
7. Validate against SOLID principles
8. Request user to review and provide feedback

## Progress Tracking

After completing design.md, update progress tracking:

### Update Notes File

Create/update `docs/coordinate/<feature>/design_notes.md`:

```markdown
# Design Notes: <Feature Name>

**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**Output**: `docs/specs/YYYY-MM-DD_<feature>/design.md`
**Date**: [YYYY-MM-DD]
**Started**: [timestamp]
**Completed**: [timestamp]
**Iterations**: [Number of design iterations]

## Architecture Summary
[High-level architecture description]

## Modules Defined
- [Module 1]: [responsibility]
- [Module 2]: [responsibility]

## Design Decisions
- [Decision 1]: [rationale]
- [Decision 2]: [rationale]

## SOLID Violations
[Any SOLID principle violations with justifications]

## Issues
[Any blockers or questions for user]

## User Feedback
[Record user approval and any comments]
```

### Output Status Report

After completion, provide:

```markdown
## Design Phase Complete

**Feature**: <feature_name>
**Status**: COMPLETED
**Output**: `docs/specs/<feature>/YYYY-MM-DD-design.md`

### Architecture
- [High-level architecture summary]

### Modules Defined
- [List key modules with responsibilities]

### Design Decisions
- [Number of key design decisions documented]

### Ready for Review
- Awaiting user approval for implementation planning
```

## Next Steps After Design

After the design.md is reviewed and approved, the next phase is **implementation_plan.md** (detailed step-by-step implementation instructions).

**You should NOT write any code in this stage.**
