---
name: review
extends: gstack-review
version: 1.0.0
description: |
  Enhanced code review with QA impact detection. Extends base gstack-review
  skill to detect test case impact during code review.
---

# review: Enhanced Code Review with QA Impact Detection

This skill EXTENDS the base `/gstack-review` skill with additional capabilities.

**Base Skill:** `/gstack-review` (symlink to gstack/review)
**Extensions:** QA impact detection, test case mapping, sync integration

---

## What QA Impact Detection Does

### 1. Maps Changed Files to Test Cases

For each file changed in the PR:
- Finds related QA test plans in `docs/qa/user-story/`
- Identifies test cases that reference the changed code
- Builds a dependency map between code and tests

### 2. Classifies Impact

| Classification | Meaning | Action Required |
|----------------|---------|-----------------|
| **BREAKING** | Code removed that test relies on | Block merge, rewrite or deprecate test |
| **NEEDS_UPDATE** | API/selector/component changed | Update test before merge |
| **NEW_TEST_NEEDED** | New feature without coverage | Generate test case |
| **OK** | No impact on tests | No action needed |

### 3. Proposes Specific Updates

For each affected test case:
- **API changes**: Shows old endpoint → new endpoint
- **Selector changes**: Shows old selector → new selector
- **Component changes**: Shows removed component → replacement
- **Flow changes**: Shows old steps → new steps

---

## Output Format

```
QA COVERAGE ISSUES
══════════════════
Found 3 issues requiring attention

1. [NEW_TEST_NEEDED] NotificationPanel component
   File: frontend/src/components/NotificationPanel.tsx
   Impact: New feature without test coverage
   Action: Generate test case: /qa --generate-test-cases NotificationPanel
   Priority: P1

2. [NEEDS_UPDATE] TC-AA-011: Usage Monitoring
   File: docs/qa/user-story/agency-cases.md:145
   Impact: API endpoint changed
   Current: /api/v1/usage/tokens
   New:     /api/v1/analytics/usage/tokens
   Action: Run scripts/qa-sync-test-case.sh TC-AA-011

3. [BREAKING] TC-TA-005: Agent Configuration
   File: docs/qa/user-story/agent-cases.md:89
   Impact: Test references removed AgentConfigDialog component
   Action: Mark test as deprecated or rewrite for new component

Recommendation: Update test cases before merging to prevent test suite rot.
```

---

## Detection Patterns

### API Endpoint Changes
```javascript
// Detect: fetch('/api/v1/usage') in test
// Detect: @router.get("/api/v1/analytics/usage") in code
// Auto-update: Update test to use /api/v1/analytics/usage
```

### Selector Changes
```javascript
// Detect: page.click('[data-testid="submit-btn"]') in test
// Detect: data-testid="confirm-submit" in changed component
// Auto-update: Update test to use [data-testid="confirm-submit"]
```

### Component Changes
```javascript
// Detect: Component removed from codebase
// Detect: Test still references removed component
// Classification: BREAKING
```

---

## Integration with Base gstack-review

QA issues are integrated into the Fix-First review workflow from base gstack-review:

1. **Auto-fix issues**: Simple updates (endpoint changes, selector changes)
2. **Ask-based issues**: Complex changes requiring user input
3. **Report-only**: Breaking changes that need manual review

QA impact detection runs automatically as Step 1.6 of the review workflow.

---

## Related Tools

| Tool | Purpose |
|------|---------|
| `/qa --sync` | Full sync workflow for test cases |
| `scripts/qa-sync-test-case.sh` | Interactive test case updates |
| `scripts/auto-update-qa-steps.js` | Auto-update MCP-Automated Test Steps |
| `qa-sync-checklist.md` | Checklist for QA sync updates |

---

## Priority Guidelines

| Change Type | Priority | Merge Block |
|-------------|----------|-------------|
| Breaking API change | P0 | Yes |
| New feature (no tests) | P1 | No |
| API endpoint change | P1 | Yes |
| Component UI change | P2 | No |
| Config change | P2 | No |

---

## Version

**Current:** 1.0.0
**Extends:** gstack-review (from gstack submodule)
**Last Updated:** 2026-05-04
**Changes:** Renamed from review-plus, now extends gstack-review explicitly
