---
name: test
description: Testing phase - write unit, integration, and E2E tests (TDD)
---

# test

You are now in the implementation phase, specifically focusing on testing. Your task is to write unit tests, integration tests, and E2E tests that cover the acceptance criteria defined in the plan.md for the current feature. Your specific instruction is: '{{args}}'. It is acceptable and even expected that at this stage some tests are failed, as we will implement code afterward to pass the tests. The failures will also be used to help understand current development progress. You should not implement the feature code in this stage, only tests.

## Test-Driven Development (TDD) Principles

**Leverage Superpowers**: This skill should be used together with `superpowers:test-driven-development` for complete TDD guidance. The superpowers skill provides detailed TDD methodology, Red-Green-Refactor cycle, and best practices for writing tests first.

Follow the TDD workflow:
1. **Red** - Write failing tests first that define expected behavior
2. **Green** - Implement minimal code to make tests pass (done in `/feature` phase)
3. **Refactor** - Improve code while keeping tests passing (done in `/feature` phase)

## Testing Layers - CRITICAL DISTINCTION

### Unit Tests
- **Purpose**: Verify individual functions/classes in isolation
- **Dependencies**: MOCKED - use unittest.mock, vitest.mock, jest.mock
- **Database**: NOT used - in-memory fake or mock
- **Network**: NOT used - mock API responses
- **Scope**: Single function, method, or component
- **Speed**: Very fast (milliseconds)

**Example unit test pattern:**
- Mock all external dependencies (API clients, database)
- Test isolated component logic
- Verify loading states, error handling, UI rendering

### Integration Tests
- **Purpose**: Verify multiple layers work together
- **Dependencies**: REAL - actual database, actual API clients
- **Database**: REAL database (test database, not production)
- **Network**: REAL HTTP calls to test API server
- **Scope**: API endpoint + database, multiple services
- **Speed**: Medium (seconds)

**Example integration test pattern:**
- Use real database connection (test database)
- Make real HTTP calls to test API server
- Verify database state matches API response

### E2E (End-to-End) Tests
- **Purpose**: Verify complete user flows through the full stack
- **Dependencies**: ALL REAL - browser, API, database
- **Browser**: REAL browser (Playwright, Cypress, Selenium)
- **API**: REAL backend API (not mocked)
- **Database**: REAL database (verify state after UI actions)
- **Scope**: Full user journey from UI → API → Database
- **Speed**: Slow (seconds to minutes)

**CRITICAL E2E REQUIREMENTS:**
1. **Real Browser**: Use Playwright/Cypress to drive actual browser
2. **Real API Calls**: Browser makes actual HTTP requests to backend
3. **Database Verification**: Query database to verify persisted state
4. **UI State Verification**: Assert UI displays data from actual API responses
5. **User Flow**: Test complete workflows (login → navigate → act → verify)

**Example E2E test pattern:**
- Real browser interaction (navigate, click, type)
- Verify UI updated with API response data
- Verify API call was made (network inspection)
- Verify database state changed (query after action)

## When to Use Each Layer

| Scenario | Test Type | Reason |
|----------|-----------|--------|
| Component renders correctly | Unit | Fast, isolated logic verification |
| Form validation logic | Unit | Edge cases for validation rules |
| API endpoint returns correct data | Integration | Real DB + API stack |
| Multiple services coordinate | Integration | Verify integration points |
| User can complete workflow | E2E | Full stack validation |
| UI displays real API data | E2E | Browser → API → UI chain |
| Database persists after UI action | E2E | Complete flow verification |

## Testing Phase Instructions

During test writing, track any issues encountered and solutions used. Maintain a mental or scratchpad list of:
- **Issues**: Test framework problems, unclear requirements, mocking challenges, environment setup issues
- **Solutions**: How each issue was resolved (test helpers, fixture updates, configuration changes, etc.)

**IMPORTANT**:
- **Generic issues**: Add to `docs/knowledge/agentic/coding/learnings.md` for reuse
- **Specific issues**: Add to `docs/implementation/<feature>/implementation_notes.md` for this feature
- Keep learning.md concise and generic
- Keep implementation_notes.md specific and actionable

## End of Testing Phase - REQUIRED

Before reporting completion, you MUST complete BOTH updates:

### 1. Update Implementation Notes

Update `docs/implementation/<feature>/implementation_notes.md` with a concise status update:

**Format to append:**

```markdown
## [Feature Name] - Phase N: Test Phase - [Date]

### Tests Written
- **test_*.py**: Description of test file
  - Test cases: X tests covering [functionality]
  - Status: ❌ Failing (TDD Red) / ✅ Passing

### Acceptance Criteria Coverage
- [ ] Criterion 1 - How it's tested
- [ ] Criterion 2 - How it's tested
- [ ] Criterion 3 - How it's tested

### Test Files Created/Modified
- `test/path/to/test_file.py`: X tests
```

### 2. Update Learnings

Update `docs/knowledge/agentic/coding/learnings.md` with ONLY generic/reusable learnings:

**Process:**
1. Read the current `docs/knowledge/agentic/coding/learnings.md` file
2. Append ONLY generic learnings (not specific to this feature)
3. Use the Edit tool to update the file
4. Then report the status and request user to review

**If no generic issues encountered**, you may skip updating learnings.md.

**Examples of GENERIC learnings:**
- Test framework patterns for async WebSocket testing
- Mocking subprocess.Popen for cross-platform tests
- Testing techniques for CDP protocol

**NOT generic (don't add to learnings.md):**
- "CDP client connection failed" (specific to CDP)
- "Browser launcher needs X flag" (specific to Chrome)

**This step is MANDATORY - do not skip implementation_notes.md update.**

## Progress Tracking

### Update Notes File

Create/update `docs/coordinate/<feature>/test_notes.md`:

```markdown
# Test Notes: <Feature Name>

## Phase N: [Phase Name]
**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**TDD Phase**: RED (tests written, expected to fail)
**Started**: [timestamp]
**Completed**: [timestamp]

### Test Results
- **Tests Written**: [N] tests
- **Tests Failing**: [M] tests (expected in RED phase)
- **Test Files**: [List of test files]

### Acceptance Criteria Coverage
| AC | Covered | Test |
|----|---------|------|
| 1. [AC description] | ✅ | [test_function] |

### Issues
[Any blockers]

### Generic Learnings
[Any reusable learnings for learnings.md]
```

### Output Status Report

After completion, provide:

```markdown
## Test Phase Complete (TDD RED)

**Feature**: <feature>
**Phase**: [N] - [Phase Name]
**Status**: COMPLETED (RED - Tests failing as expected)

### Test Results
- **Tests Written**: [N] tests
- **Tests Failing**: [M] tests (expected in TDD RED phase)
- **Test Files**: [List of test files]

### Acceptance Criteria Coverage
| AC | Covered | Test |
|----|---------|------|
| 1. [AC description] | ✅ | [test_function] |
| 2. [AC description] | ✅ | [test_function] |

### Next Steps
- Ready for feature implementation phase
- Feature engineer should write code to make tests pass
```

## Test Files Created

Track all test files created:

```
test/unit_test/<path>/test_<module>.py: [N] tests
test/integration_test/<path>/test_<module>_integration.py: [N] tests
```

## Blocking Issues

If blocked during test writing:
1. Document the issue in `docs/implementation/<feature>/implementation_notes.md`
2. Update progress file with BLOCKED status
3. Specify what is needed to unblock
4. **DO NOT WAIT** - report the blocker and move on

## E2E Test Requirements - FULL STACK VALIDATION

**CRITICAL**: E2E tests MUST validate the complete flow from user interaction through the API to database persistence.

### Required E2E Test Coverage

Every E2E test should verify at minimum:

1. **UI Interaction** - Real browser action (click, type, navigate)
2. **API Request** - Verify real HTTP request was made to backend
3. **API Response** - Verify backend responds with correct data
4. **UI Update** - Verify UI displays the API response data
5. **Database State** - Verify database reflects the change

### E2E Test Anti-Patterns (DO NOT DO)

❌ **BAD: Only console.log checks** - Doesn't verify actual behavior
❌ **BAD: Mocking API responses in E2E** - This makes it a UNIT test, not E2E
❌ **BAD: Only checking UI elements exist** - Doesn't verify functionality
❌ **BAD: No database verification** - Missing critical layer of validation

### E2E Test Best Practices

✅ **GOOD: Complete flow verification**
- Perform real UI action (click button, type in input)
- Verify UI updates (element appears/disappears, text changes)
- Verify API call was made (network inspection, status code)
- Verify API response contains expected data
- Verify database state changed (query after action)

### E2E Test Structure

**Test Pattern:**
- ARRANGE: Setup test data in database
- ACT: UI interaction (navigate, click, type)
- ASSERT: Multi-layer verification
  - UI updated immediately
  - API call was made with correct data
  - Database state changed

### Database Verification in E2E Tests

E2E tests MUST verify database state. Use one of these approaches:

**Option 1: Direct database connection (preferred)**
- Create a database connection for verification
- Query database to verify persisted state
- Most reliable method

**Option 2: Admin API for verification (if DB not accessible)**
- Use admin API to verify persisted state
- Less reliable than direct DB access

**Option 3: Re-query via UI (least reliable)**
- Refresh page and verify state persists
- May have timing issues

## E2E Test File Organization

```
frontend/src/__tests__/
├── unit/                    # Unit tests - mocked dependencies
│   └── components/
│       └── admin/
│           └── ResourcesManagement.test.tsx
├── integration/             # Integration tests - real API, real DB
│   └── api/
│       └── admin/
│           └── resources.test.ts
└── e2e/                     # E2E tests - full stack, real browser
    └── admin_resources.spec.ts
```

## Test Coverage Matrix

| Feature | Unit Tests | Integration Tests | E2E Tests |
|---------|------------|-------------------|-----------|
| Component rendering | ✅ | ❌ | ❌ |
| Form validation | ✅ | ❌ | ❌ |
| API endpoint logic | ❌ | ✅ | ❌ |
| Database queries | ❌ | ✅ | ❌ |
| User flows | ❌ | ❌ | ✅ |
| UI displays API data | ❌ | ❌ | ✅ |
| Database persistence | ❌ | ✅ | ✅ |

## Writing E2E Tests - Step by Step

### Step 1: Define User Flow
- Map out complete user journey
- Include all UI interactions
- Identify expected system responses at each layer

### Step 2: Write Test with Full Stack Verification
- Setup: Create test data in database
- Login/Auth as needed
- Navigate to feature
- Perform UI action
- Verify UI updated
- Verify API called
- Verify database changed

### Step 3: Add Edge Cases
- Unauthorized access attempts
- Idempotent behavior
- Side effects and cascading changes
- Concurrent operations
- Network failures
- Boundary conditions
