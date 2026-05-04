# QA Test Case Sync Checklist

This checklist is used by the code-reviewer agent to verify that QA test cases stay in sync with implementation changes.

---

## Before Updating Test Cases

### Verify Intent
- [ ] Confirm the implementation change is intentional (not a bug)
- [ ] Check if test case should be deprecated (removed) instead of updated
- [ ] Verify the new behavior is correct before updating tests

### Check Test Plan Relevance
- [ ] Does the test plan still apply to the current feature?
- [ ] Has the feature evolved beyond the original test scope?
- [ ] Are there edge cases not covered by existing tests?

---

## When Updating Test Cases

### Test Steps Table Updates
- [ ] Update `Action` column with new UI interactions/API calls
- [ ] Update `Expected Output` column to match new behavior
- [ ] Update `Decision Strategy` with new verification logic
- [ ] Ensure step numbering is sequential
- [ ] Check for orphaned references to removed components

### Acceptance Criteria Updates
- [ ] Update AC to reflect new feature requirements
- [ ] Remove deprecated acceptance criteria
- [ ] Add new acceptance criteria for added functionality
- [ ] Ensure AC is verifiable and testable

### MCP-Automated Test Steps Updates
- [ ] Update API endpoint URLs in fetch/page.goto calls
- [ ] Update CSS selectors in page.click/page.fill calls
- [ ] Update aria-label references
- [ ] Update assertion logic to match new behavior
- [ ] Test the JavaScript code syntax (no syntax errors)

### Verification Result Section
- [ ] Update execution timestamp
- [ ] Update environment information
- [ ] Document what changed and why
- [ ] Note any manual testing performed

---

## Creating New Test Cases

### Test Case Structure
- [ ] Follow TC-XXX naming convention (TC-AA-###, TC-TA-###, etc.)
- [ ] Include descriptive title after ID
- [ ] Add Test Steps table with all required columns
- [ ] Add Acceptance Criteria list (numbered)
- [ ] Add MCP-Automated Test Steps section
- [ ] Include Edge Cases table (if applicable)
- [ ] Include Security Considerations (if applicable)

### Test Step Format
- [ ] Step column: Sequential numbers (1, 2, 3...)
- [ ] Action column: Clear, actionable instructions
- [ ] Expected Output column: Observable, measurable results
- [ ] Decision Strategy column: Verification logic (URL is X, count > 0, etc.)

### MCP-Automated Test Steps Format
- [ ] Wrap in ```javascript ... ``` code block
- [ ] Use async/await for all page interactions
- [ ] Include page.goto for navigation steps
- [ ] Use specific selectors (data-testid, aria-label)
- [ ] Include assertions for verification
- [ ] Add error handling (try/catch for optional elements)

---

## After Updates

### Verification
- [ ] Run `/qa-plus` to verify updated test case passes
- [ ] Check for JavaScript syntax errors in MCP steps
- [ ] Verify selectors actually exist in the application
- [ ] Test manually if automation fails

### Documentation
- [ ] Commit with conventional commit message: `test(qa): sync TC-XXX with implementation changes`
- [ ] Reference related issue or PR in commit body
- [ ] Update CHANGELOG if test case requirements changed significantly

### Review
- [ ] Have another developer review the test case changes
- [ ] Ensure test plan document is still readable and clear
- [ ] Check for typos or formatting issues
- [ ] Verify test case ID uniqueness

---

## Common Update Scenarios

### API Endpoint Changes
```
Before: await page.goto('http://localhost:8001/api/v1/usage')
After:  await page.goto('http://localhost:8001/api/v1/analytics/usage')
Action: Update all fetch/page.goto calls with new endpoint
```

### Component Name Changes
```
Before: Click "AgentSkillSelector" button
After:  Click "SkillSelector" button
Action: Update selector references in test steps
```

### UI Structure Changes
```
Before: Fill "agent-name" input
After:  Fill input[placeholder="Agent name"]
Action: Update selector to use more stable identifier
```

### Flow Changes
```
Before: Navigate to /agents → Click New → Fill form → Submit
After:  Navigate to /agents/skills → Click Add Skill → Configure → Save
Action: Update test steps to match new user flow
```

---

## Test Case Deprecation

When deprecating a test case instead of updating:

1. Add deprecation notice:
   ```markdown
   ## TC-XXX-001: Old Feature Test

   **DEPRECATED**: This test case is deprecated as of 2026-05-02.
   **Reason**: Feature removed in favor of New Feature (TC-XXX-015).
   **Migration**: Use TC-XXX-015 instead.
   ```

2. Move to archive directory:
   ```bash
   mkdir -p docs/qa/user-story/deprecated
   mv docs/qa/user-story/old-feature-cases.md docs/qa/user-story/deprecated/
   ```

3. Update index if one exists

---

## Auto-Update Patterns

### Selector Detection
When reviewing code changes, detect these patterns:
- `data-testid="..."` changes → Update test selectors
- `aria-label="..."` changes → Update test selectors  
- Component rename → Update test step descriptions

### API Detection
When reviewing code changes, detect these patterns:
- FastAPI route decorator changes → Update API endpoint tests
- Request/response schema changes → Update test assertions
- Query parameter changes → Update test URLs

### Flow Detection
When reviewing code changes, detect these patterns:
- Redirect changes → Update test navigation steps
- Form field additions/removals → Update test step tables
- Multi-step flow changes → Update test step sequences

---

## Integration with Code Review

The code-reviewer agent should:

1. **Map files to test cases**: For each changed file, find related test cases
2. **Detect breaking changes**: Identify tests that reference removed code
3. **Propose updates**: Suggest specific changes to test steps/selectors/APIs
4. **Flag gaps**: Note new features without test coverage
5. **Generate test stubs**: Create template test cases for new features

### Output Format for Code Review

```
QA COVERAGE ISSUES
══════════════════
[NEW_TEST_NEEDED] components/NotificationPanel.tsx
  Impact: New feature without test coverage
  Action: Generate test case: /qa-plus --generate-test-cases NotificationPanel
  Priority: P1

[NEEDS_UPDATE] TC-AA-011: Usage Monitoring
  Impact: API endpoint changed
  Current: /api/v1/usage/tokens
  New:     /api/v1/analytics/usage/tokens
  File: docs/qa/user-story/agency-cases.md:145
  Action: Run scripts/qa-sync-test-case.sh TC-AA-011

[BREAKING] TC-TA-005: Agent Configuration
  Impact: Test references removed AgentConfigDialog component
  File: docs/qa/user-story/agent-cases.md:89
  Action: Mark test as deprecated or rewrite for new component
```

---

## Test Case Quality Checklist

### Clarity
- [ ] Is the test purpose clear from the title?
- [ ] Are steps unambiguous and actionable?
- [ ] Is expected output observable and measurable?

### Completeness
- [ ] Does the test cover the happy path?
- [ ] Does the test cover error cases?
- [ ] Does the test cover edge cases?
- [ ] Are acceptance criteria comprehensive?

### Maintainability
- [ ] Are selectors stable (data-testid vs CSS classes)?
- [ ] Is the test independent of other tests?
- [ ] Can the test be run in isolation?
- [ ] Is the test automation code readable?

### Reusability
- [ ] Can test steps be reused across similar test cases?
- [ ] Are common patterns abstracted?
- [ ] Is there a page object for the feature?
