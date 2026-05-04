---
name: feature
description: Feature implementation phase with TDD - write code to pass tests
---

# feature

You are now in the implementation phase, specifically focusing on feature development. Your task is to write the code for the feature to make all existing unit and integration tests pass. Your specific instruction is: '{{args}}'. Adhere strictly to the design.md and plan.md documents. Iterate on tests and code until all acceptance criteria are satisfied.

## Test-Driven Development (TDD) - MANDATORY

You MUST follow TDD principles:
1. **Red** - Tests are already written (from `/test` phase) and currently failing
2. **Green** - Write ONLY the minimal code needed to make tests pass
3. **Refactor** - Improve code structure while keeping tests green

**IMPORTANT**: Do NOT write code without tests. Do NOT skip tests. All features must have:
- Unit tests for all functions/classes
- Integration tests for interactions
- E2E tests for end-to-end workflows
- Tests covering acceptance criteria from plan.md

**Leverage Superpowers**: This skill should be used together with `superpowers:test-driven-development` for full TDD guidance. The superpowers skill provides detailed TDD methodology, test-first practices, and the Red-Green-Refactor cycle.

**E2E Tests**:
- Must be written in `/test` phase before `/feature` phase
- Should cover complete user workflows (e.g., multi-tab scenarios, real-world usage)
- Should validate integration of all components working together
- Should be documented in `test/e2e/<feature>/test_<feature>_steps.md`
- Should be referenced in implementation_plan.md for the corresponding phase
- Tests should be in "Red" state (failing or skip) before `/feature` phase begins

## Implementation Phase Instructions

During implementation, track any issues encountered and solutions used. Maintain a mental or scratchpad list of:
- **Issues**: Bugs, errors, blockers, unexpected behaviors, design conflicts
- **Solutions**: How each issue was resolved (code changes, configuration updates, refactoring, etc.)

## Verification Checklist - MANDATORY

Before reporting completion, you MUST verify:

1. **All Tests Pass**
    - Run: `python scripts/test/run_tests.py test unit`
    - Run: `python scripts/test/run_integration_tests.py` (if applicable)
    - Run E2E tests: `test/e2e/<feature>/test_<feature>_steps.md` (verify scenarios manually or via automation)
    - For tests with external dependencies (database, LLM, Docker, etc.), run with appropriate markers:
      - `python scripts/test/run_integration_tests.py --markers docker` (for Docker tests)
      - `python scripts/test/run_integration_tests.py --markers database` (for database tests)
      - `python scripts/test/run_integration_tests.py --markers llm` (for LLM tests)
    - NO failing tests allowed

2. **Code Quality**
   - Code follows SOLID principles from `docs/knowledge/agentic/coding/engineering.md`
   - No obvious code smells or technical debt
   - Proper error handling

3. **Documentation**
   - Code is self-documenting with clear names
   - Complex logic has inline comments
   - API changes are documented

## End of Implementation - REQUIRED

Before reporting completion, you MUST:

1. **Update implementation notes**: Update `docs/implementation/<feature>/implementation_notes.md` with a concise summary of changes made in this phase, mention next steps.

2. **Document learnings**: If any issue is met and resolved in a generic way (not limited to this feature and phase), make a concise summary in `docs/knowledge/agentic/coding/learnings.md`.

3. **Verify all tests pass**:
    - Unit: `python scripts/test/run_tests.py test unit`
    - Integration: `python scripts/test/run_integration_tests.py` (if applicable)
    - E2E: Manual or automated verification of scenarios in `test/e2e/<feature>/test_<feature>_steps.md`
    - NO failing tests allowed

4. **Commit changes** with descriptive title and comprehensive description.

5. **Push changes** to remote git.

**ALL steps are MANDATORY - do not skip any.**

## Progress Tracking

### Update Notes File

Create/update `docs/coordinate/<feature>/feature_notes.md`:

```markdown
# Feature Notes: <Feature Name>

## Phase N: [Phase Name]
**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**TDD Phase**: GREEN (implementation complete, tests passing)
**Started**: [timestamp]
**Completed**: [timestamp]

### Implementation Results
- **Files Modified**: [List of implementation files]
- **Tests Before**: [N] failing
- **Tests After**: [N] passing (100%)
- **Commit**: [commit_hash]

### Acceptance Criteria Met
| AC | Status | Evidence |
|----|--------|----------|
| 1. [AC description] | ✅ | [test_function] |

### Issues
[Any blockers]

### Generic Learnings
[Any reusable learnings for learnings.md]
```

### Output Status Report

After completion, provide:

```markdown
## Feature Phase Complete (TDD GREEN)

**Feature**: <feature>
**Phase**: [N] - [Phase Name]
**Status**: COMPLETED (GREEN - All tests passing)

### Implementation Results
- **Files Modified**: [List of files]
- **Lines Added**: [N]
- **Tests Before**: [M] failing
- **Tests After**: [M] passing ✅

### Acceptance Criteria Met
| AC | Status | Evidence |
|----|--------|----------|
| 1. [AC description] | ✅ | [test_function] |
| 2. [AC description] | ✅ | [test_function] |

### Commit Info
- **Hash**: [commit_hash]
- **Message**: [commit message]

### Next Steps
- Ready for integration testing or next phase
- All tests passing, code ready for review
```

## Implementation Files Created/Modified

Track all implementation files:

```
src/<path>/<module>.py: [description of changes]
src/<path>/<module>.py: [description of changes]
```

## Blocking Issues

If blocked during implementation:
1. Document the issue in `docs/implementation/<feature>/implementation_notes.md`
2. Update progress file with BLOCKED status
3. Specify what is needed to unblock
4. **DO NOT WAIT** - report the blocker and move on
