---
name: plan
description: Planning stage - create/update implementation plan.md
---

# plan

You are now in the planning stage for the feature. Your task is to draft (or update if it exists) the `implementation_plan.md` file. Your specific instruction is: `'{{args}}'`.

## Purpose

The **implementation_plan.md** document provides **detailed step-by-step implementation instructions**. This is the most detailed document, containing specific files, code structure, acceptance criteria, and testing strategies for each phase.

**Leverage Superpowers**: This skill should be used together with `superpowers:writing-plans` for comprehensive implementation planning guidance. The superpowers skill provides detailed methodology for breaking down requirements into actionable implementation steps.

## Documentation Location

`docs/implementation/<feature_name>/implementation_plan.md`

## Content Requirements

Your implementation_plan.md must include:

### 1. Implementation Phases

Break the work into small, testable phases. Each phase should include:

#### Phase N: [Phase Name]

**Tasks:**
1. [Specific task with file paths]
2. [Specific task with file paths]
3. [Specific task with file paths]

**Acceptance Criteria:**
- [ ] [Criterion 1 - verifiable]
- [ ] [Criterion 2 - verifiable]
- [ ] [Criterion 3 - verifiable]

**Testing:**
- **Unit Tests**: What to test at unit level
- **Integration Tests**: Component interactions to test
- **Test Files**: Where tests should be located

**Dependencies:**
- Blocks: [Phases that depend on this]
- Blocked by: [Phases this depends on]

**Risks:**
- [Potential risk and mitigation]

### 2. Code Structure

Document the file and directory structure:
```
src/
  feature/
    module_a.py
    module_b.py
test/
  unit_test/
    feature/
      test_module_a.py
      test_module_b.py
```

### 3. Code Examples and Pseudo-code

Include helpful code snippets, interface definitions, or pseudo-code when needed to clarify implementation details.

## Guidelines

**Break down into small, testable phases:**
- Each phase should be completable in a reasonable time
- Each phase must have clear acceptance criteria
- Each phase must be testable independently

**Be specific:**
- ✅ Include file paths
- ✅ Include function/class names
- ✅ Include code snippets when helpful
- ✅ Define exact acceptance criteria

**Define testing strategy:**
- Unit tests for each function/class
- Integration tests for component interactions
- E2E tests for user scenarios (document in `test/e2e/test_<feature>_steps.md`)

## Process

1. Read the instruction: `'{{args}}'`
2. Review spec.md and design.md to understand the feature
3. Break down implementation into phases
4. Define acceptance criteria for each phase
5. Outline testing strategy (unit, integration, E2E)
6. Identify dependencies and risks
7. Include code examples where helpful
8. Request user to review and provide feedback

## Documentation Workflow

```
1. spec.md (WHAT)
   ↓
2. design.md (HOW - Architecture)
   ↓
3. implementation_plan.md (HOW - Detailed Steps) ← You are here
   ↓
4. Implement following phases
   ↓
5. Update status as work progresses
```

## Progress Tracking

After completing implementation_plan.md, update progress tracking:

### Update Notes File

Create/update `docs/coordinate/<feature>/plan_notes.md`:

```markdown
# Plan Notes: <Feature Name>

**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**Output**: `docs/implementation/<feature>/implementation_plan.md`
**Started**: [timestamp]
**Completed**: [timestamp]
**Phases Defined**: [Number of phases]
**Total Tasks**: [Total tasks across all phases]

## Phase Summary
| Phase | Tasks | ACs | Dependencies |
|-------|-------|-----|--------------|
| 1. [Name] | [N] | [M] | [None/Phase X] |
| 2. [Name] | [N] | [M] | [Phase 1] |

## Issues
[Any blockers or questions for user]

## User Feedback
[Record user approval and any comments]
```

### Output Status Report

After completion, provide:

```markdown
## Plan Phase Complete

**Feature**: <feature_name>
**Status**: COMPLETED
**Output**: `docs/implementation/<feature>/implementation_plan.md`

### Implementation Plan
- **Phases**: [N] phases defined
- **Tasks**: [M] total tasks
- **Estimated complexity**: [Low/Medium/High]

### Phase Summary
| Phase | Tasks | ACs | Dependencies |
|-------|-------|-----|--------------|
| 1. [Name] | [N] | [M] | [None/Phase X] |
| 2. [Name] | [N] | [M] | [Phase 1] |
| ... | ... | ... | ... |

### Ready for Review
- Awaiting user approval for TDD implementation
- Ready to start test phase
```

## Next Steps After Plan

After the implementation_plan.md is reviewed and approved, the next phase is **implementation** following the defined phases.

**You should NOT write any implementation code in this stage.**
