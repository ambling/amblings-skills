---
name: review-and-fix
description: Code review with automatic fixes based on project standards
---

# review-and-fix

Review code changes following the project review guide, report on test and documentation coverage, and automatically fix issues found.

**User's instruction:** {{args}}

## Overview

This skill performs automated code review based on `@docs/knowledge/agentic/coding/how-to-review.md`, checking current changes against project standards, running tests, reporting coverage gaps, and fixing issues automatically.

**Leverage Superpowers**: When receiving code review feedback or addressing review comments, use `superpowers:receiving-code-review` to systematically evaluate feedback before implementing changes.

## Your Task

1. Identify what changed (git diff, target branch)
2. Run the review checklist from the guide
3. Execute tests and report results
4. Check documentation coverage
5. Fix identified issues automatically
6. Provide comprehensive report

## Approach

### Step 1: Identify Changes

```bash
# Get current branch and changes
git branch --show-current
git status
git diff --stat
git log --oneline -5

# Identify target branch (usually main or origin/main)
git log --oneline --all --graph -10
```

Extract:
- **Files changed** - What was modified/added/deleted
- **Change type** - Feature, bug fix, refactoring, docs
- **Scope** - Which module/feature is affected
- **Target branch** - Base branch for comparison
- **Languages** - Python, JavaScript/TypeScript, or both

### Step 2: Pre-Review Setup

```bash
# Ensure virtual environment is active (for Python)
source .venv/bin/activate  # or activate via your preferred method

# Get baseline test status
python scripts/test/run_tests.py test unit 2>&1 | head -50

# Check if node_modules exists for JavaScript/TypeScript
[ -d "ui/node_modules" ] && echo "Node deps present" || echo "Need npm install"
```

### Step 2.5: Run Code Style Lint and Format (AUTOMATIC FIX)

**CRITICAL: Always run linting and formatting FIRST before any other review steps.**

```bash
# Python: Run ruff format (auto-fixes formatting issues)
ruff format backend test scripts

# Python: Run ruff linting (auto-fixes what it can)
ruff check backend test scripts --fix

# JavaScript/TypeScript: Run prettier (if applicable)
npx prettier --write "ui/**/*.{js,jsx,ts,tsx,json,css,scss}"

# JavaScript/TypeScript: Run eslint (if applicable)
npx eslint ui/**/*.{js,jsx,ts,tsx} --fix
```

**After auto-fix, check remaining issues:**
```bash
# Python: Check for remaining ruff issues
ruff check backend test scripts

# JavaScript: Check for remaining eslint issues
npx eslint ui/**/*.{js,jsx,ts,tsx}
```

**If any remaining issues:**
1. Fix critical ones automatically
2. Document non-critical ones in the report
3. Add specific fix recommendations

### Step 3: Review Checklist Execution

Follow `@docs/knowledge/agentic/coding/how-to-review.md` checklist:

**Documentation & Requirements:**
- [ ] Spec exists for new features
- [ ] Changelog updated
- [ ] Breaking changes documented
- [ ] E2E test steps document (for nontrivial features)

**Code Quality:**
- [ ] PEP 8 compliant (check with ruff/pylint)
- [ ] Type hints present
- [ ] Meaningful names
- [ ] Proper error handling
- [ ] No hardcoded values
- [ ] No code duplication

**Testing:**
- [ ] Unit tests written (>80% coverage)
- [ ] Integration tests updated
- [ ] E2E tests added (for user-facing features)
- [ ] Edge cases covered
- [ ] Error paths tested

**Security & Safety:**
- [ ] No secrets in code
- [ ] Input validation
- [ ] SQL injection prevention
- [ ] User data isolation
- [ ] Sensitive data handling

### Step 4: Run Tests and Coverage

```bash
# Backend unit tests
python scripts/test/run_tests.py test unit

# Integration tests
docker compose -f docker-compose.test.yml up -d
python scripts/test/run_integration_tests.py

# With specific markers
python scripts/test/run_integration_tests.py --markers database
python scripts/test/run_integration_tests.py --markers llm
python scripts/test/run_integration_tests.py --markers oauth

# Check coverage
python -m pytest test/unit_test/ --cov=src --cov-report=term-missing

# Type checking
mypy src/

# Python linting (after formatting)
ruff check src/
pylint src/

# JavaScript linting (if applicable)
npx eslint ui/

# Security scan
bandit -r src/
```

### Step 5: Documentation Coverage Check

Check for:
- [ ] Feature spec: `docs/specs/{feature}/spec.md`
- [ ] Design doc: `design.md` (if significant)
- [ ] Implementation plan: `docs/specs/{feature}/implementation_plan.md`
- [ ] E2E test steps: `test/e2e/test_{feature}_steps.md`
- [ ] API docs updated (if API changes)
- [ ] README updated (if user-facing)
- [ ] Changelog entry: `docs/CHANGELOG.md`

### Step 6: Fix Issues Automatically

**IMPORTANT: Code style fixes are already done in Step 2.5. Focus on other issues.**

**Fix in order of priority:**

1. **Critical fixes** (blocking):
   - Failing tests → Fix test logic or implementation
   - Security issues → Remove secrets, add validation
   - Import errors → Fix imports or dependencies
   - Syntax errors → Fix code

2. **Important fixes**:
   - Type hints missing → Add type annotations
   - Error handling missing → Add try/except
   - Input validation missing → Add validation
   - Hardcoded values → Move to config/env

3. **Code quality fixes**:
   - Meaningless names → Rename variables/functions
   - Code duplication → Extract common logic
   - Missing docstrings → Add documentation
   - Remaining linting issues → Fix (after ruff format --fix)

4. **Test coverage fixes**:
   - Missing unit tests → Add test cases
   - Missing integration tests → Add integration tests
   - Edge cases not covered → Add test cases

5. **Documentation fixes**:
   - Missing spec → Create spec.md
   - Missing E2E steps → Create test steps document
   - Changelog not updated → Add entry

### Step 7: Re-verify After Fixes

```bash
# Run tests again
python scripts/test/run_tests.py test unit

# Check if fixes work
python scripts/test/run_integration_tests.py

# Verify coverage improved
python -m pytest test/unit_test/ --cov=src --cov-report=term-missing
```

## Input Format

```
/review-and-fix
/review-and-fix focus on {module/feature}
/review-and-fix check {file-path}
/review-and-fix for branch {branch-name}
```

**Examples:**
```
/review-and-fix
/review-and-fix focus on if-accountant skill
/review-and-fix check src/agent/skills/if-database/
/review-and-fix for feature/user-authentication
```

## Output Format

Provide a comprehensive report:

```markdown
# Code Review Report

## Summary
- **Branch:** {branch_name}
- **Target:** {target_branch}
- **Files changed:** {count}
- **Change type:** {feature/bugfix/refactor/docs}

## Changes Overview
{List of files changed with brief descriptions}

## Review Checklist Results

### Documentation & Requirements
- [x] Spec exists
- [ ] Changelog updated
- [ ] E2E test steps document
...

### Code Quality
- [x] PEP 8 compliant
- [x] Type hints present
- [ ] Proper error handling
...

### Testing
- [x] Unit tests written
- [ ] Integration tests updated
- [ ] E2E tests added
...

### Security & Safety
- [x] No secrets in code
- [x] Input validation
...

## Test Results

### Unit Tests
```
{Test output summary}
```

### Integration Tests
```
{Test output summary}
```

### Coverage Report
```
{Coverage summary}
```

## Issues Found

### Critical (1)
{Issue description}

### Important (2)
{Issue descriptions}

### Code Quality (3)
{Issue descriptions}

## Fixes Applied

### Fixed Issues
1. {What was fixed}
2. {What was fixed}

### Files Modified
{List of files modified during fixes}

## Re-verification Results

### Tests After Fixes
{Test results}

### Coverage After Fixes
{Coverage results}

## Recommendations

{Suggestions for further improvements}

## Next Steps
{What to do next}
```

## Examples

### Example 1: Full Review

**User input:**
```
/review-and-fix focus on if-accountant skill
```

**Output:**
```
# Code Review Report

## Summary
- **Branch:** feature/accountant-updates
- **Target:** main
- **Files changed:** 5
- **Change type:** Feature

## Changes Overview
- Modified: src/agent/skills/if-accountant/SKILL.md
- Modified: src/agent/skills/if-accountant/scripts/extract_account.py
- Added: src/agent/skills/if-accountant/scripts/match_accounts.py
- Modified: workspace/accountant/IDENTITY.md
- Added: workspace/accountant/TOOLS.md

## Review Checklist Results

### Documentation & Requirements
- [x] SKILL.md updated
- [x] IDENTITY.md created
- [x] TOOLS.md updated
- [ ] Changelog not updated
- [ ] E2E test steps document missing

### Code Quality
- [x] PEP 8 compliant (checked with ruff)
- [ ] Type hints missing in extract_account.py
- [x] Meaningful names
- [ ] Proper error handling
- [ ] No hardcoded values
- [x] No code duplication

### Testing
- [x] Unit tests exist
- [ ] Unit tests pass (2/3 passing)
- [ ] Integration tests updated
- [ ] E2E tests missing

## Issues Found

### Critical (0)
None

### Important (2)
1. Type hints missing in extract_account.py functions
2. One failing unit test: test_extract_account.py::test_mask_account_number

### Code Quality (1)
1. Missing docstring in match_accounts.py

## Fixes Applied

### Fixed Issues
1. Added type hints to all functions in extract_account.py
2. Fixed test_mask_account_number assertion (was checking wrong format)
3. Added module docstring to match_accounts.py

### Files Modified
- src/agent/skills/if-accountant/scripts/extract_account.py
- test/unit_test/agent/skills/test_extract_account.py
- src/agent/skills/if-accountant/scripts/match_accounts.py

## Re-verification Results

### Tests After Fixes
```
Unit tests: 15/15 passing (100%)
Integration tests: 8/8 passing (100%)
```

### Coverage After Fixes
```
Name                                    Stmts   Miss  Cover
-----------------------------------------------------
src/agent/skills/if-accountant/           145      8    94%
```

## Recommendations

1. Add changelog entry to docs/CHANGELOG.md
2. Create E2E test steps document
3. Consider adding more edge case tests for OCR failures

## Next Steps

- [ ] Update changelog
- [ ] Create test/e2e/test_accountant_steps.md
- [ ] Add OCR error handling tests
- [ ] Commit fixes with message: "fix(accountant): add type hints and fix test"
```

### Example 2: Quick Check

**User input:**
```
/review-and-fix check src/agent/skills/if-database/scripts/db_accounts.py
```

**Output:**
```
# Code Review Report

## Summary
- **File:** src/agent/skills/if-database/scripts/db_accounts.py
- **Status:** Ready to merge

## Review Results

### Code Quality
- [x] PEP 8 compliant
- [x] Type hints present
- [x] Meaningful names
- [x] Proper error handling
- [x] No hardcoded values
- [x] No code duplication

### Security
- [x] No secrets in code
- [x] Input validation present
- [x] User data isolation

### Testing
- [x] Unit tests exist
- [x] Tests passing

## Issues Found

None - file follows all project standards.

## Test Results

```
test/unit_test/agent/skills/test_if_database_accounts.py::test_get_accounts - PASSED
test/unit_test/agent/skills/test_if_database_accounts.py::test_create_account - PASSED
test/unit_test/agent/skills/test_if_database_accounts.py::test_update_account - PASSED
```

## Status: ✅ No issues found
```

## Review Priority Matrix

When multiple issues found, fix in this order:

1. **Critical** - Tests failing, security issues, syntax errors
2. **Important** - Type hints, error handling, input validation
3. **Quality** - Code style, naming, duplication
4. **Coverage** - Missing tests, edge cases
5. **Documentation** - Missing specs, changelog, E2E steps

## Auto-Fix Capabilities

This skill can automatically fix:

| Issue Type | Fix Method |
|------------|------------|
| Code style (Python) | `ruff format` + `ruff check --fix` |
| Code style (JS/TS) | `prettier --write` + `eslint --fix` |
| Failing tests | Update test assertions or fix implementation |
| Missing type hints | Add type annotations to functions |
| Missing error handling | Add try/except blocks |
| Hardcoded values | Move to config/env |
| Import errors | Fix import paths |
| Missing docstrings | Add module/function docstrings |
| Code duplication | Extract common logic |

## Manual Review Required

These issues need human judgment:
- Architecture decisions
- Algorithm choices
- Performance trade-offs
- API design changes
- Breaking changes (need migration guide)
- Security edge cases

## Related Documentation

- Review guide: `@docs/knowledge/agentic/coding/how-to-review.md`
- Engineering principles: `@docs/knowledge/agentic/coding/engineering.md`
- Testing guide: Refer to specific skill/component documentation
- Project structure: `@docs/knowledge/agentic/coding/engineering.md` (Organization section)

## Common Fix Patterns

### Add Type Hints
```python
# Before
def get_balance(account_id):
    result = api_call(f"/accounts/{account_id}/balance")
    return result

# After
def get_balance(account_id: str) -> dict[str, Any]:
    result = api_call(f"/accounts/{account_id}/balance")
    return result
```

### Add Error Handling
```python
# Before
def extract_data(file_path):
    content = Path(file_path).read_text()
    return process(content)

# After
def extract_data(file_path: str) -> dict[str, Any]:
    try:
        content = Path(file_path).read_text()
    except FileNotFoundError:
        return {"error": f"File not found: {file_path}"}
    except Exception as e:
        return {"error": f"Failed to read file: {e}"}
    return process(content)
```

### Remove Hardcoded Values
```python
# Before
API_URL = "http://localhost:8000"
TIMEOUT = 30

# After
API_URL = os.getenv("IF_BACKEND_URL", "http://localhost:8000")
TIMEOUT = int(os.getenv("API_TIMEOUT", "30"))
```

## Progress Tracking

After review completion, update progress tracking:

### Update Notes File

Create/update `docs/coordinate/<feature>/review_notes.md`:

```markdown
# Review Notes: <Feature Name>

## Phase N: [Phase Name] (or full feature review)
**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]
**Started**: [timestamp]
**Completed**: [timestamp]

### Review Results
- **Critical Issues**: [N] found, [M] fixed
- **Important Issues**: [N] found, [M] fixed
- **Quality Issues**: [N] found, [M] fixed
- **Tests**: [All passing / X failing]
- **Coverage**: [percentage]%

### Fixes Applied
- [List of fixes applied]

### Commit
- **Hash**: [commit_hash]
- **Message**: [commit message]

### Remaining Issues
[Any issues not fixed, with reason]

### Issues
[Any blockers]
```

### Output Status Report

After completion, provide:

```markdown
## Review Phase Complete

**Feature**: <feature>
**Phase**: [N] - [Phase Name] (or full feature review)
**Status**: COMPLETED

### Review Results
- **Critical Issues**: [N] found, [M] fixed
- **Important Issues**: [N] found, [M] fixed
- **Quality Issues**: [N] found, [M] fixed
- **Tests**: [All passing ✅ / X failing]
- **Coverage**: [percentage]%

### Fixes Applied
- [List of fixes applied]

### Commit Info
- **Hash**: [commit_hash]
- **Message**: [commit message]

### Remaining Issues
- [Any issues not fixed, with reason]

### Next Steps
- Ready for next phase / QA / merge
```

## Blocking Issues

If blocked during review (e.g., critical architectural issues):
1. Document the issue in detail
2. Update progress file with BLOCKED status
3. Recommend specific actions to resolve
4. **DO NOT WAIT** - report the blocker and move on
```
