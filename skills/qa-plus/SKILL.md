# qa-plus: Enhanced QA with Test Plan Execution & E2E Integration

Extends the base `/qa` skill to:
1. Execute test plans from Markdown files in `docs/qa/user-story/`
2. Run existing E2E tests and include results
3. Update Verification Result sections automatically
4. Generate comprehensive test reports

---

## Usage

### Basic Full QA (original behavior)
```bash
/qa-plus http://localhost:5173
```
Runs systematic exploration like `/qa`.

### Execute Test Plan
```bash
/qa-plus http://localhost:5173 --test-plan docs/qa/user-story/agency-and-agent-management-cases.md
```
Parses Markdown test cases and executes them systematically.

### Run with E2E Tests
```bash
/qa-plus http://localhost:5173 --with-e2e
```
Runs full QA + executes E2E test suite.

### Complete QA (Test Plan + E2E)
```bash
/qa-plus http://localhost:5173 --test-plan docs/qa/user-story/agency-and-agent-management-cases.md --with-e2e
```

### Local Docker Build Test (NEW)
```bash
/qa-plus --local
```
Builds a fresh Docker image and runs QA against it locally. Exposes Docker build issues before PR merge.

**What it does:**
1. Builds frontend (`npm run build`)
2. Builds Docker image from current code
3. Runs container locally on port 8000
4. Executes full QA suite against local container
5. Cleans up container after testing

**Why use this:**
- Catches Docker container startup failures (like PYTHONPATH issues)
- Validates that production Docker image works correctly
- Tests against exact production environment
- Prevents broken deployments

**Parameters for local mode:**
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--local` | false | Enable local Docker build test mode |
| `--keep-container` | false | Keep container running after QA (for debugging) |
| `--container-port` | 8000 | Port to run container on |

---

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Target URL | Required (unless --local) | Application URL to test |
| `--local` | false | Build and test Docker image locally |
| `--test-plan` | None | Path to Markdown test plan file |
| `--with-e2e` | false | Run E2E tests after manual QA |
| `--tier` | Standard | Quick/Standard/Exhaustive |
| `--output` | .gstack/qa-reports/ | Report output directory |
| `--sync` | false | Sync test cases with implementation changes |
| `--generate-test-cases` | None | Generate test cases for new features |

---

## Test Plan Format

This skill parses Markdown test cases following the ClawFarm format:

```markdown
## TC-AA-001: Agency Registration

| Step | Action | Expected Output | Decision Strategy |
|------|--------|-----------------|-------------------|
| 1 | Navigate to /register | Page loads | URL is /register |
| 2 | Fill registration form | Form accepts input | No validation errors |
| 3 | Click submit | Redirects to dashboard | URL becomes /dashboard |

## Acceptance Criteria
1. User receives agency ID
2. Dashboard shows agency name
3. User is logged in
```

---

## Sync Mode: Keeping Test Cases Current

Use `--sync` to automatically detect and update test cases when implementation changes:

```bash
/qa-plus http://localhost:5173 --sync
```

**What sync mode does:**

1. **Detects affected test cases** - Maps changed files to related test plans
2. **Classifies impact** - BREAKING, NEEDS_UPDATE, NEW_TEST_NEEDED, or OK
3. **Proposes updates** - Shows specific changes needed in test steps/selectors/APIs
4. **Generates diffs** - Shows before/after for proposed changes
5. **Applies updates** - Updates test plans after user confirmation

### Sync Output Format

```
QA SYNC ANALYSIS
══════════════════
Analyzing 3 changed files against 12 test plans...

Affected Test Cases: 5
  [BREAKING]    TC-TA-005: Agent Configuration
    Reason: Component AgentConfigDialog removed
    Files: frontend/src/components/AgentConfigDialog.tsx (deleted)
    Action: Mark as deprecated or rewrite

  [NEEDS_UPDATE] TC-AA-011: Usage Monitoring  
    Reason: API endpoint changed
    Current: /api/v1/usage/tokens?period=this_month
    New:     /api/v1/analytics/usage/tokens?period=this_month
    File: docs/qa/user-story/agency-cases.md:145 (MCP Test Step)
    
    Proposed Change:
    -   await fetch('/api/v1/usage/tokens?period=' + period);
    +   await fetch('/api/v1/analytics/usage/tokens?period=' + period);
    
    Apply? A) Yes  B) Skip  C) Edit manually

  [NEW_TEST_NEEDED] NotificationPanel component
    Reason: New feature without test coverage
    File: frontend/src/components/NotificationPanel.tsx (added)
    Action: Generate test case with /qa-plus --generate-test-cases NotificationPanel

Sync Summary: 1 breaking, 3 needs_update, 1 new_test_needed
Recommendation: Update test cases before merging to prevent test suite rot.
```

### Auto-Update MCP-Automated Test Steps

The sync mode can automatically detect and update MCP-Automated Test Steps when implementation changes:

#### Detection Patterns

**API Endpoint Changes:**
```javascript
// Detect: fetch('/api/v1/usage') in test
// Detect: @router.get("/api/v1/analytics/usage") in code
// Auto-update: Update test to use /api/v1/analytics/usage
```

**Selector Changes:**
```javascript
// Detect: page.click('[data-testid="submit-btn"]') in test
// Detect: data-testid="confirm-submit" in changed component
// Auto-update: Update test to use [data-testid="confirm-submit"]
```

**Flow Changes:**
```javascript
// Detect: Multi-step flow in test
// Detect: New/removed navigation step in component
// Auto-update: Reorder or update test steps
```

#### Auto-Update Process

1. **Parse MCP-Automated Test Steps** from test case
2. **Extract URLs, selectors, and assertions** from JavaScript code
3. **Search for changes** in related implementation files
4. **Generate mapping** from old → new values
5. **Apply updates** with user confirmation
6. **Verify syntax** of updated JavaScript code

#### Example Auto-Update

```javascript
// Before (test case):
await page.goto('http://localhost:8001/api/v1/usage/tokens');
await page.click('[data-testid="agent-submit"]');

// After (auto-updated):
await page.goto('http://localhost:8001/api/v1/analytics/usage/tokens');
await page.click('[data-testid="agent-config-save"]');
```

### Generate Test Cases Mode

Create test cases for new features:

```bash
/qa-plus http://localhost:5173 --generate-test-cases NotificationPanel
```

This analyzes the component and generates a test case template following the ClawFarm QA format.

---

## Workflow

### Phase 0: Local Docker Build (if --local flag)

This phase builds and tests the application in a Docker container that mirrors production.

**Steps:**

1. **Pre-flight Checks**
   ```bash
   # Check Docker is available
   docker --version
   docker ps
   ```

2. **Build Frontend**
   ```bash
   cd frontend
   npm run build
   # Verify frontend/dist exists
   ```

3. **Copy Static Files to Backend**
   ```bash
   rm -rf backend/static
   cp -r frontend/dist backend/static
   ```

4. **Build Docker Image**
   ```bash
   docker build -t clawfarm-qa:latest .
   ```

5. **Run Container Locally**
   ```bash
   # Set required environment variables
   export DATABASE_URL="postgresql://test:test@localhost:5432/test" || true
   export REDIS_URL="redis://localhost:6379" || true

   # Run container (ports: 8000 for app, 5432 for test DB if needed)
   docker run -d \
     --name clawfarm-qa \
     -p 8000:8000 \
     -e PYTHONPATH=/app \
     -e DATABASE_URL="$DATABASE_URL" \
     -e REDIS_URL="$REDIS_URL" \
     -e ENVIRONMENT=test \
     clawfarm-qa:latest
   ```

6. **Wait for Container to Start**
   ```bash
   # Wait up to 30 seconds for health endpoint
   for i in {1..30}; do
     if curl -sf http://localhost:8000/health > /dev/null 2>&1; then
       echo "✓ Container is healthy"
       break
     fi
     sleep 1
   done
   ```

7. **Check Container Logs**
   ```bash
   docker logs clawfarm-qa
   ```

8. **If container fails to start:**
   - Capture full container logs
   - Check for Python import errors
   - Check for missing dependencies
   - Report Docker build failure immediately
   - **Exit with error code 10** (Docker build failure)

9. **Set QA Target to Local Container**
   - Override target URL to `http://localhost:8000`
   - Continue with standard QA phases

10. **Cleanup (unless --keep-container)**
    ```bash
    docker stop clawfarm-qa
    docker rm clawfarm-qa
    docker rmi clawfarm-qa:latest
    ```

**Error Codes for Local Mode:**

| Code | Meaning |
|------|---------|
| 10 | Docker build failed |
| 11 | Container failed to start |
| 12 | Container not healthy after 30s |
| 13 | Frontend build failed |

**Common Issues This Detects:**

1. **Missing PYTHONPATH** - Container can't import Python modules
2. **Missing dependencies** - Requirements not properly installed
3. **Wrong CMD/ENTRYPOINT** - Container starts but app doesn't run
4. **Port binding issues** - App listening on wrong port
5. **Static file issues** - Frontend not properly copied
6. **Environment variable issues** - Missing required env vars

**Example Output:**

```
=== Local Docker Build QA ===
Building frontend...
✓ Frontend built in 2.3s

Building Docker image...
✓ Image built: clawfarm-qa:latest (sha256:abc123...)

Starting container...
✓ Container running as clawfarm-qa (port 8000)

Waiting for health check...
.....✓ Health endpoint responding

Running QA against http://localhost:8000...
[QA continues...]
```

### Phase 1: Initialize
1. Parse command parameters
2. Create output directories
3. Load test plan if specified
4. Start timer

### Phase 1.5: Sync Analysis (if --sync flag)
1. Get list of changed files from git diff
2. Find affected test plans in `docs/qa/user-story/`
3. For each affected test plan:
   - Parse test case IDs and content
   - Map changed files to test case references
   - Classify impact: BREAKING, NEEDS_UPDATE, NEW_TEST_NEEDED, or OK
4. Generate proposed changes for each affected test case:
   - Extract current values from test steps/MCP steps
   - Detect new values from changed files
   - Create before/after diff for review
5. Present findings to user with action options:
   - A) Apply all proposed updates
   - B) Review each update individually
   - C) Generate new test cases for missing coverage
   - D) Skip (manual review later)
6. Apply approved updates to test plan files
7. Generate sync report with all changes made

### Phase 2: Test Plan Execution (if --test-plan)
1. Parse Markdown file
2. Extract test cases (## TC-XXX headers)
3. For each test case:
   - Parse Test Steps table
   - Extract actions, expected outputs, decision strategies
   - Execute steps with Playwright
   - Verify against acceptance criteria
   - Take screenshots
   - Calculate pass/fail
4. Update Verification Result sections in Markdown

### Phase 3: Manual QA Exploration
1. Navigate to target URL
2. Take initial screenshot
3. Map navigation structure
4. Visit pages systematically
5. Check console errors
6. Document issues

### Phase 4: E2E Test Execution (if --with-e2e)
1. Check for E2E test configuration
2. Run E2E tests:
   ```bash
   cd frontend && npm run test:e2e
   ```
3. Parse results
4. Include in QA report:
   - Pass/fail counts
   - Failed test names
   - Screenshot links

### Phase 5: Report Generation
1. Compute health score
2. Generate comprehensive report with:
   - Manual QA findings
   - Test plan results (if applicable)
   - E2E test results (if applicable)
   - Overall health score
   - Recommendations

---

## Test Plan Execution Details

### Parsing Markdown Test Cases

The skill extracts:

```bash
# For each ## TC-XXX section:
- Test case ID and name
- Priority (from header or metadata)
- Test Steps table (Step, Action, Expected Output, Decision Strategy)
- Acceptance Criteria list
- Edge Cases table (if present)
- Security Considerations (if present)
```

### Executing Test Steps

Actions are mapped to Playwright commands:

| Action Pattern | Playwright Command |
|----------------|-------------------|
| `Navigate to /path` | `page.goto('/path')` |
| `Click [element]` | `page.click(selector)` |
| `Fill [field]` | `page.fill(selector, value)` |
| `Verify [condition]` | `expect(condition).toBeTruthy()` |
| `Screenshot` | `page.screenshot(path)` |

### Decision Strategies

The "Decision Strategy" column is used to verify pass/fail:

```
Example: "URL is /dashboard"
→ Check: page.url() === 'http://localhost:5173/dashboard'

Example: "No validation errors"
→ Check: page.locator('.error').count() === 0
```

---

## Updating Verification Results

After executing a test case, the skill updates the Markdown file:

```markdown
## Verification Result

**Executed**: 2026-05-02 14:30 by /qa-plus (automated)
**Environment**: Local (http://localhost:5173)
**Duration**: 2 minutes 15 seconds

- **Passed Steps**: 3 / 3
- **Passed ACs**: 3 / 3
- **Overall**: 100%

### Pass/Fail Criteria
- **PASS**: All 3 acceptance criteria met, all steps passed

**Result**: PASS

### Evidence
- Screenshot: `.gstack/qa-reports/test-plans/TC-AA-001-registration-success.png`
- Console: 0 errors
- Network: 0 failed requests

### Issues Found
None
```

---

## E2E Integration

### Test Discovery

The skill looks for E2E tests in:

```bash
frontend/src/__tests__/e2e/
```

### Execution

```bash
cd frontend
npm run test:e2e --reporter=json --reporter=./e2e-results.json
```

### Result Parsing

The skill parses Playwright JSON output:

```json
{
  "stats": {
    "expected": 15,
    "unexpected": 1,
    "flaky": 0,
    "skipped": 2
  }
}
```

### Report Section

```markdown
## E2E Test Results

**Command:** `npm run test:e2e`
**Duration:** 3 minutes 45 seconds
**Framework:** Playwright

### Summary
- **Total Tests:** 18
- **Passed:** 15 (83%)
- **Failed:** 1 (6%)
- **Skipped:** 2 (11%)

### Failed Tests
1. `admin_page_structure.spec.ts:23` - Admin tabs not loading (timeout)

### Flaky Tests
None detected

### Skipped Tests
1. `vertical-onboarding.spec.ts` - Feature not enabled
2. `frontend-serving.spec.ts` - Requires production URL
```

---

## Output Structure

```
.gstack/qa-reports/
├── qa-report-localhost-5173-2026-05-02-comprehensive.md
├── test-plans/
│   ├── TC-AA-001-registration-success.png
│   ├── TC-AA-003-agent-creation.png
│   └── ...
├── e2e/
│   ├── e2e-results.json
│   └── screenshots/
└── baseline-comprehensive.json
```

---

## Example Report Section

```markdown
## Test Plan Results: Agency and Agent Management

**Source:** `docs/qa/user-story/agency-and-agent-management-cases.md`
**Test Cases:** 47
**Executed:** 12 (P0 only)
**Duration:** 8 minutes

### Summary
| Priority | Total | Passed | Failed | Skipped |
|----------|-------|--------|--------|---------|
| P0 | 21 | 18 | 2 | 1 |
| P1 | 18 | - | - | - |
| P2 | 8 | - | - | - |

### Passed Tests
- TC-AA-001: Agency Registration ✓
- TC-AA-003: Agent Creation ✓
- TC-AA-007: LLM Provider Selection ✓
- ... (15 more)

### Failed Tests
- TC-AA-005: Agent Skill Assignment
  - Step 4 failed: Skill dropdown not populating
  - Screenshot: test-plans/TC-AA-005-failure.png
  - Root cause: API endpoint returning 500

- TC-AA-011: Usage Monitoring
  - AC 2 failed: Real-time dashboard not updating
  - Screenshot: test-plans/TC-AA-011-failure.png
  - Root cause: WebSocket connection failing

### Overall Test Plan Score: 85%
```

---

## Configuration

### E2E Test Configuration

Create `frontend/.qa-e2e.config.json`:

```json
{
  "enabled": true,
  "command": "npm run test:e2e",
  "timeout": 300000,
  "reporter": "json",
  "outputDir": ".gstack/qa-reports/e2e"
}
```

### Test Plan Mapping

Create `docs/qa/.test-plan-mapping.json`:

```json
{
  "agency-and-agent-management-cases.md": {
    "priority": "P0",
    "estimatedTime": "8 minutes",
    "testType": "functional"
  },
  "channel-integration-cases.md": {
    "priority": "P1",
    "estimatedTime": "12 minutes",
    "testType": "integration",
    "requiresCredentials": true
  }
}
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All tests passed |
| 1 | Critical failures found |
| 2 | Test plan execution failed |
| 3 | E2E tests failed |
| 10 | Docker build failed (--local mode) |
| 11 | Container failed to start (--local mode) |
| 12 | Container not healthy after 30s (--local mode) |
| 13 | Frontend build failed (--local mode) |

---

## Version

**Current:** 1.1.0
**Based on:** `/qa` skill from gstack
**Last Updated:** 2026-05-02
**Changes:** Added `--local` mode for Docker build testing
