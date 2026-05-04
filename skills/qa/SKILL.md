---
name: qa
extends: gstack-qa
version: 1.1.0
description: |
  Extended QA with test plan execution, E2E integration, and sync mode.
  Extends base gstack-qa skill with additional capabilities.
---

# qa: Extended QA Features

This skill EXTENDS the base `/gstack-qa` skill with additional capabilities.

**Base Skill:** `/gstack-qa` (symlink to gstack/qa)
**Extensions:** Test plan execution, E2E integration, sync mode, Docker local testing

---

## Usage

### Basic Full QA (base gstack-qa behavior)
```bash
/qa http://localhost:5173
```
Runs systematic exploration using base gstack-qa skill.

### Execute Test Plan
```bash
/qa http://localhost:5173 --test-plan docs/qa/user-story/agency-and-agent-management-cases.md
```
Parses Markdown test cases and executes them systematically.

### Run with E2E Tests
```bash
/qa http://localhost:5173 --with-e2e
```
Runs full QA + executes E2E test suite.

### Complete QA (Test Plan + E2E)
```bash
/qa http://localhost:5173 --test-plan docs/qa/user-story/agency-and-agent-management-cases.md --with-e2e
```

### Local Docker Build Test
```bash
/qa --local
```
Builds a fresh Docker image and runs QA against it locally.

**What it does:**
1. Builds frontend (`npm run build`)
2. Builds Docker image from current code
3. Runs container locally on port 8000
4. Executes full QA suite against local container
5. Cleans up container after testing

---

## Custom Parameters Added

These parameters EXTEND the base gstack-qa parameters:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--test-plan` | None | Path to Markdown test plan file |
| `--with-e2e` | false | Run E2E tests after QA |
| `--local` | false | Build and test Docker image locally |
| `--sync` | false | Sync test cases with implementation |
| `--generate-test-cases` | None | Generate test cases for new features |

---

## Custom Workflow Extensions

### Phase 1.5: Sync Analysis (if --sync flag)

The sync mode automatically detects and updates test cases when implementation changes:

```bash
/qa http://localhost:5173 --sync
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
    Action: Generate test case with /qa --generate-test-cases NotificationPanel

Sync Summary: 1 breaking, 3 needs_update, 1 new_test_needed
Recommendation: Update test cases before merging to prevent test suite rot.
```

### Phase 2: Test Plan Execution (if --test-plan)

Parses Markdown test cases following the ClawFarm format and executes systematically:

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

### Phase 4: E2E Test Execution (if --with-e2e)

```bash
cd frontend
npm run test:e2e --reporter=json --reporter=./e2e-results.json
```

Includes results in QA report:
- Pass/fail counts
- Failed test names
- Screenshot links

---

## Local Docker Build Details

### Error Codes

| Code | Meaning |
|------|---------|
| 10 | Docker build failed |
| 11 | Container failed to start |
| 12 | Container not healthy after 30s |
| 13 | Frontend build failed |

### Common Issues Detected

1. **Missing PYTHONPATH** - Container can't import Python modules
2. **Missing dependencies** - Requirements not properly installed
3. **Wrong CMD/ENTRYPOINT** - Container starts but app doesn't run
4. **Port binding issues** - App listening on wrong port
5. **Static file issues** - Frontend not properly copied
6. **Environment variable issues** - Missing required env vars

---

## Integration with Base gstack-qa

Base `/gstack-qa` workflow runs first. This skill's custom phases inject at specific points:

| Custom Phase | Injection Point |
|--------------|-----------------|
| Sync analysis | After initialization, before exploration |
| Test plan execution | Replaces exploration phase |
| E2E execution | After manual QA |
| Local Docker | Replaces entire target URL setup |

---

## Version

**Current:** 1.1.0
**Extends:** gstack-qa (from gstack submodule)
**Last Updated:** 2026-05-04
**Changes:** Renamed from qa-plus, now extends gstack-qa explicitly
