# QA Coverage Check Specialist

This specialist reviews code changes for QA test case coverage impact and synchronization needs.

---

## Mission

Detect when implementation changes make QA test cases outdated or missing, and propose specific updates to keep test documentation in sync.

---

## Detection Patterns

### 1. New Feature Detection

**Pattern**: Files added that don't match test file patterns

```bash
# Find new non-test files
NEW_FILES=$(git diff origin/<base>...HEAD --name-only | grep "^A" | grep -vE "\.(test|spec)\.(tsx?|jsx?|py)$|\.config\.(js|ts)")

for FILE in $NEW_FILES; do
  # Check if test plan mentions this file
  if ! grep -qr "$(basename $FILE)" docs/qa/user-story/; then
    echo "NEW_TEST_NEEDED: $FILE - No test coverage found"
  fi
done
```

**Severity**: INFORMATIONAL
**Action**: Propose generating test case with `/qa-plus --generate-test-cases`

---

### 2. Component/UI Changes

**Pattern**: Significant changes to `.tsx`, `.jsx`, `.vue` files

```javascript
// Check for structural changes in components
const componentChanges = changedFiles.filter(f => 
  f.match(/\.(tsx|jsx|vue)$/) && 
  gitStats[f].insertions > 50 || gitStats[f].deletions > 50
);

for (const file of componentChanges) {
  // Find test cases that reference this component
  const affectedTests = findTestsReferencingFile(file);
  
  for (const test of affectedTests) {
    // Check if test uses selectors that might be affected
    const selectors = extractSelectors(test.mcpSteps);
    const changedSelectors = findChangedSelectors(file, selectors);
    
    if (changedSelectors.length > 0) {
      console.log(`NEEDS_UPDATE: ${test.id}`);
      console.log(`  Affected selectors: ${changedSelectors.join(', ')}`);
    }
  }
}
```

**Severity**: INFORMATIONAL (if changes < 100 lines), CRITICAL (if changes > 100 lines)

**Checklist Items**:
- Does test use CSS classes that were changed?
- Does test use data-testid attributes that were removed?
- Does test rely on aria-labels that were modified?
- Does test reference component names that changed?

---

### 3. API Endpoint Changes

**Pattern**: Changes to FastAPI route decorators or API response schemas

```bash
# Find changed API routes
grep -rn "^@router\|^@app\|^@.*\.route\(" backend/api/v1/ --include="*.py" | \
  git diff origin/<base>...HEAD --name-only | \
  xargs -I {} sh -c 'echo "Changed routes in:"; git diff origin/<base>...HEAD {} | grep "^+.*route"'

# For each changed endpoint, find test cases that use it
grep -r "'/api/v1/[^']*'" docs/qa/user-story/ | while read line; do
  PLAN=$(echo "$line" | cut -d: -f1)
  if git diff origin/<base>...HEAD --name-only | grep -q "$(basename $(echo "$line" | cut -d: -f1))"; then
    echo "CHECK_ENDPOINT: $PLAN uses changed endpoint"
  fi
done
```

**Severity**: CRITICAL (API contract changed)

**Checklist Items**:
- Did endpoint URL change (path restructure)?
- Did request method change (GET → POST)?
- Did request/response schema change?
- Did authentication requirements change?
- Did error responses change?

---

### 4. Database Schema Changes

**Pattern**: Changes to SQLAlchemy models or Alembic migrations

```bash
# Find changed model files
CHANGED_MODELS=$(git diff origin/<base>...HEAD --name-only | grep "models/.*\.py$")

for MODEL in $CHANGED_MODELS; do
  # Find test cases that reference this model
  grep -r "$(basename $MODEL .py)" docs/qa/user-story/ | while read line; do
    PLAN=$(echo "$line" | cut -d: -f1)
    echo "CHECK_MODEL: $PLAN may reference changed model $MODEL"
  done
done
```

**Severity**: CRITICAL (if columns removed), INFORMATIONAL (if columns added)

**Checklist Items**:
- Did column names change?
- Were columns removed or renamed?
- Did column types change (affecting validation)?
- Did relationships change?
- Did constraints change (unique, nullable, etc.)?

---

### 5. Configuration Changes

**Pattern**: Changes to feature flags, environment variables, config files

```bash
# Find changed config
CONFIG_CHANGES=$(git diff origin/<base>...HEAD --name-only | grep -E "config\.|\.env\|feature")

for CONFIG in $CONFIG_CHANGES; do
  # Check if test cases reference these configs
  grep -r "$(basename $CONFIG)" docs/qa/user-story/ | while read line; do
    PLAN=$(echo "$line" | cut -d: -f1)
    echo "CHECK_CONFIG: $PLAN may depend on $CONFIG"
  done
done
```

**Severity**: INFORMATIONAL

**Checklist Items**:
- Did feature flag default change?
- Did environment variable name change?
- Did configuration structure change?
- Did default values change?

---

## Finding Schema

```json
{
  "severity": "CRITICAL|INFORMATIONAL",
  "confidence": 1-10,
  "path": "file_path",
  "line": null,
  "category": "qa-sync",
  "summary": "Short description of QA impact",
  "fix": "Recommended action to fix",
  "fingerprint": "test_case_id:category",
  "specialist": "qa-coverage",
  "test_stub": "Suggested test case content if new test needed",
  "affected_tests": ["TC-XXX-001", "TC-XXX-005"],
  "change_type": "BREAKING|NEEDS_UPDATE|NEW_TEST_NEEDED",
  "current_value": "What the test currently has",
  "new_value": "What the test should have"
}
```

---

## Output Examples

### New Feature Without Tests

```json
{
  "severity": "INFORMATIONAL",
  "confidence": 8,
  "path": "frontend/src/components/NotificationPanel.tsx",
  "line": null,
  "category": "qa-sync",
  "summary": "New NotificationPanel component has no test coverage",
  "fix": "Generate test case: /qa-plus --generate-test-cases NotificationPanel",
  "fingerprint": "NEW_TEST:NotificationPanel",
  "specialist": "qa-coverage",
  "test_stub": "## TC-NF-001: Notification Panel Display\n\n| Step | Action | Expected Output | Decision Strategy |\n|------|--------|-----------------|-------------------|\n| 1 | Navigate to /dashboard | Page loads with notification bell | URL is /dashboard |\n| 2 | Click notification bell icon | Panel opens | Panel is visible |\n\n## Acceptance Criteria\n1. Notification bell is visible in header\n2. Clicking bell shows notification list\n3. Notifications are grouped by date",
  "change_type": "NEW_TEST_NEEDED"
}
```

### API Endpoint Changed

```json
{
  "severity": "CRITICAL",
  "confidence": 9,
  "path": "docs/qa/user-story/agency-cases.md",
  "line": 145,
  "category": "qa-sync",
  "summary": "Test case TC-AA-011 references changed API endpoint",
  "fix": "Update MCP-Automated Test Step: change '/api/v1/usage/tokens' to '/api/v1/analytics/usage/tokens'",
  "fingerprint": "TC-AA-011:api-change",
  "specialist": "qa-coverage",
  "affected_tests": ["TC-AA-011"],
  "change_type": "NEEDS_UPDATE",
  "current_value": "await fetch('/api/v1/usage/tokens?period=this_month')",
  "new_value": "await fetch('/api/v1/analytics/usage/tokens?period=this_month')"
}
```

### Component Removed

```json
{
  "severity": "CRITICAL",
  "confidence": 10,
  "path": "docs/qa/user-story/agent-cases.md",
  "line": 89,
  "category": "qa-sync",
  "summary": "Test case TC-TA-005 references deleted AgentConfigDialog component",
  "fix": "Mark test as deprecated or rewrite to use new ConfigurationModal component",
  "fingerprint": "TC-TA-005:breaking",
  "specialist": "qa-coverage",
  "affected_tests": ["TC-TA-005"],
  "change_type": "BREAKING",
  "current_value": "Click 'AgentConfigDialog' button",
  "new_value": null
}
```

---

## Classification Logic

```javascript
function classifyImpact(change, testCase) {
  // BREAKING: Code removed that test relies on
  if (change.type === 'deleted' && testCase.references.includes(change.file)) {
    return 'BREAKING';
  }
  
  // NEEDS_UPDATE: Code changed in ways that affect test
  if (change.type === 'modified' && testCase.hasReferencesTo(change.file)) {
    if (change.hasStructuralChanges) {
      return 'NEEDS_UPDATE';
    }
  }
  
  // NEW_TEST_NEEDED: New feature code added
  if (change.type === 'added' && change.isFeatureFile) {
    if (!testCase.existsFor(change.file)) {
      return 'NEW_TEST_NEEDED';
    }
  }
  
  // OK: No impact on test
  return 'OK';
}
```

---

## Integration with Review Workflow

The QA Coverage specialist findings are merged into the main review and presented to the user:

```
QA COVERAGE ISSUES
══════════════════
Found 3 issues requiring attention

1. [NEW_TEST_NEEDED] NotificationPanel component
   File: frontend/src/components/NotificationPanel.tsx
   Action: Generate test case with /qa-plus --generate-test-cases
   
2. [NEEDS_UPDATE] TC-AA-011: Usage Monitoring  
   File: docs/qa/user-story/agency-cases.md:145
   Current: /api/v1/usage/tokens
   New: /api/v1/analytics/usage/tokens
   Action: Run scripts/qa-sync-test-case.sh TC-AA-011
   
3. [BREAKING] TC-TA-005: Agent Configuration
   File: docs/qa/user-story/agent-cases.md:89
   Reason: References deleted AgentConfigDialog component
   Action: Mark as deprecated or rewrite test

Recommendation: Update test cases before merging to prevent test suite rot.
```

---

## Priority Guidelines

| Change Type | Priority | Action Required |
|-------------|----------|-----------------|
| Breaking API change | P0 | Block merge until tests updated |
| New feature (no tests) | P1 | Generate tests or create TODO |
| API endpoint change | P1 | Update tests before merge |
| Component UI change | P2 | Update selectors when convenient |
| Config change | P2 | Update if test relies on config |
| Documentation change | P3 | Optional |

---

## Test Stub Templates

### UI Component Test

```markdown
## TC-UI-XXX: [Component Name] Display

| Step | Action | Expected Output | Decision Strategy |
|------|--------|-----------------|-------------------|
| 1 | Navigate to [page] | Page loads | URL contains [path] |
| 2 | Locate [component] | Component is visible | element.querySelector('[data-testid="..."]') exists |
| 3 | [Interaction] | [Result] | [Verification] |

## Acceptance Criteria
1. [Component] renders correctly
2. [Interaction] works as expected
3. [Edge case] is handled

## MCP-Automated Test Steps

\`\`\`javascript
await page.goto('http://localhost:5173/[path]');
const component = await page.locator('[data-testid="..."]');
await expect(component).toBeVisible();
// [Interaction code]
\`\`\`
```

### API Endpoint Test

```markdown
## TC-API-XXX: [Endpoint Name]

| Step | Action | Expected Output | Decision Strategy |
|------|--------|-----------------|-------------------|
| 1 | Send request to [endpoint] | Returns expected data | Response status is 200 |
| 2 | Verify response schema | Data matches expected format | JSON schema validation passes |

## Acceptance Criteria
1. Endpoint returns correct data
2. Error cases return appropriate error codes
3. Authentication is enforced
```
