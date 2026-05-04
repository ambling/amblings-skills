---
name: setup-project
description: Establish comprehensive project documentation, testing processes, and development workflow standards
---

# setup-project

Establish comprehensive project documentation (CLAUDE.md), testing infrastructure, documentation structure, and development workflow standards for a codebase.

**User's instruction:** {{args}}

## Your Task

Transform an existing codebase into a well-documented, process-driven project with:

1. **CLAUDE.md**: Central AI development guide
2. **Documentation Structure**: Organized docs/ directory with specs, bugs, QA, coordination
3. **Testing Infrastructure**: Unit/integration/E2E test patterns and requirements
4. **Development Processes**: Pre-commit testing, bug tracking workflow, feature size guidelines
5. **Helper Scripts**: Makefile, bug report generator, pre-commit test runner

## Approach

### Phase 1: Explore the Codebase

Understand the current state before making changes:

1. **Analyze Testing Setup**
   - Backend testing framework (pytest, unittest, jest, vitest?)
   - Frontend testing framework
   - E2E testing setup (Playwright, Cypress?)
   - Test coverage and quality
   - Test file locations and naming conventions

2. **Analyze Documentation**
   - Existing README, docs/ structure
   - API documentation patterns
   - Bug tracking (if any)
   - Progress tracking methods

3. **Analyze Development Workflow**
   - Git workflow (branches, PRs, direct commits?)
   - Commit message patterns
   - Code review processes
   - Feature development patterns

4. **Analyze Project Structure**
   - Directory organization
   - Tech stack and architecture
   - Key patterns and conventions

### Phase 2: Create CLAUDE.md

Create a comprehensive CLAUDE.md in the project root with:

1. **Project Overview**
   - Tech stack
   - Architecture highlights
   - Key directories and their purposes

2. **Development Workflow**
   - When to use coordinate-dev-workflow vs direct implementation
   - Feature size criteria (small vs big)
   - Commit standards

3. **Testing Requirements**
   - Backend testing requirements
   - Frontend testing requirements
   - E2E requirements matrix by change type
   - Pre-commit checklist

4. **Bug Documentation**
   - Bug lifecycle (NEW → IN_PROGRESS → VERIFIED → FIXED → CLOSED)
   - Bug template structure
   - Where to document bugs

5. **Critical Patterns to Reuse**
   - Common backend patterns
   - Common frontend patterns
   - E2E test patterns

6. **Anti-Patterns to Avoid**
   - Common mistakes in this codebase

7. **Quick Reference**
   - Common commands
   - File locations
   - Getting help

### Phase 3: Establish Documentation Structure

Create or organize the `docs/` directory:

```
docs/
├── specs/              # Feature specifications
├── design/             # Design documents
├── qa/                 # QA test cases and results
│   ├── cases/          # Test case files
│   └── results/        # Test execution reports
├── coordinate/         # Team coordination docs
│   ├── bugs/           # Bug reports
│   └── ACTIVE-COORDINATION.md
└── implementation/     # Implementation plans
```

Create directory structure if it doesn't exist.

### Phase 4: Create Helper Scripts and Tooling

1. **Bug Report Generator** (`scripts/create-bug.sh`)
   - Auto-increment bug IDs
   - Use standardized template
   - Make executable

2. **Pre-commit Test Runner** (`scripts/precommit-tests.sh`)
   - Run fast unit tests
   - Fail if tests don't pass
   - Clear output

3. **Makefile** (project root)
   - Common development commands
   - Unified interface for tasks
   - Include: dev, test, build, bug, docs targets

4. **E2E Test Templates** (`{test-dir}/e2e/TEMPLATES.md`)
   - Common test patterns
   - Helper functions
   - Best practices

### Phase 5: Update Package Configuration

Add pre-commit test scripts to package.json or equivalent:

```json
"scripts": {
  "test:precommit": "run fast tests only"
}
```

### Phase 6: Commit and Verify

1. Stage all new files
2. Commit with conventional commit message
3. Verify scripts are executable
4. Test the new processes

## Input Format

```
setup-project [--minimal | --full]
```

- **No flags**: Standard setup with all phases
- **--minimal**: CLAUDE.md and basic structure only
- **--full**: Complete setup including all templates and helper scripts

## Output Format

Creates/updates the following files:

```
CLAUDE.md                              # Project guide
Makefile                               # Development commands
scripts/create-bug.sh                  # Bug generator
scripts/precommit-tests.sh             # Test runner
{test-dir}/e2e/TEMPLATES.md            # E2E patterns
docs/coordinate/bugs/.gitkeep          # Bug directory
```

## Examples

### Basic Usage

```
/setup-project
```

Establishes full project documentation and processes.

### Minimal Setup

```
/setup-project --minimal
```

Creates CLAUDE.md and basic docs structure only.

### For Different Project Types

**Python Backend:**
```
/setup-project
```
Creates pytest-based testing patterns.

**React Frontend:**
```
/setup-project
```
Creates vitest + Playwright patterns.

**Full Stack:**
```
/setup-project
```
Creates both backend and frontend patterns.

## CLAUDE.md Template Reference

The CLAUDE.md should include:

### Project Overview (5-10 lines)
- Tech stack
- Architecture highlights
- Key directories

### Development Workflow (10-15 lines)
- Small vs Big feature criteria
- When to use coordinate-dev-workflow
- Direct implementation guidelines

### Testing Requirements (15-20 lines)
- Backend testing framework and requirements
- Frontend testing framework and requirements
- E2E requirements matrix
- Coverage thresholds

### Bug Documentation (5-10 lines)
- Bug lifecycle
- Bug template
- Location for bug reports

### Commit Standards (5-10 lines)
- Conventional commit format
- When to add tests

### Critical Patterns (10-15 lines)
- Common patterns to reuse
- File locations with `@` references

### Anti-Patterns (5-10 lines)
- Common mistakes to avoid

### Quick Reference (10-15 lines)
- Common commands
- File locations
- Help resources

## Testing Requirements Matrix Template

| Change Type | Unit Tests | Integration Tests | E2E Tests |
|-------------|-----------|-------------------|-----------|
| Bug fix | ✅ Required | If backend logic | If UI affected |
| New component | ✅ Required | If API calls | ✅ Required |
| API endpoint | ✅ Required | ✅ Required | If UI uses it |
| Admin feature | ✅ Required | ✅ Required | ✅ Required |
| Styling only | ❌ Optional | ❌ Optional | ✅ Screenshot verification |

## Bug Report Template

```markdown
# BUG-XXX: [Title]

**Severity**: [MEDIUM/HIGH/CRITICAL]
**Status**: NEW
**Found During**: [Test case / feature name]
**Date Found**: [YYYY-MM-DD]

## Summary
[Brief description]

## Steps to Reproduce
1. Step one
2. Step two

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Impact
[Business and technical impact]

## Root Cause
[Analysis of the issue]

## Recommended Fix
[Recommended approach to fix]

## Test Verification
After fix, verify:
1. Step one
2. Step two

## Related Test Cases
- TC-XXX

## Files Likely Requiring Changes
- File path 1
- File path 2
```

## Helper Script Templates

### Bug Report Generator (`scripts/create-bug.sh`)

Key features:
- Find highest existing bug ID
- Create new bug file with incremented ID
- Prompt for title, severity, summary
- Use standardized template
- Make executable with `chmod +x`

### Pre-commit Test Runner (`scripts/precommit-tests.sh`)

Key features:
- Run unit tests only (fast feedback)
- Backend: `pytest test/unit/ --cov --cov-fail-under=80`
- Frontend: `npm run test:unit`
- Fail if any test fails
- Clear colored output

### Makefile Targets

Required targets:
- `make install` - Install dependencies
- `make dev` - Start development server
- `make test` - Run all tests
- `make precommit` - Run pre-commit tests
- `make lint` - Run linters
- `make build` - Build project
- `make bug` - Create bug report
- `make docs` - View CLAUDE.md
- `make help` - Show available commands

## Directory Creation

Create these directories if they don't exist:

```bash
mkdir -p docs/specs
mkdir -p docs/design
mkdir -p docs/qa/cases
mkdir -p docs/qa/results
mkdir -p docs/coordinate/bugs
mkdir -p docs/implementation
mkdir -p scripts
```

Add `.gitkeep` files to empty directories to preserve them in git.

## Commit Message Template

```
docs(process): establish project-wide development processes and CLAUDE.md

This commit establishes comprehensive development processes to ensure
consistent, high-quality development with proper test coverage.

Created Files:
- CLAUDE.md: Central AI development guide
- Makefile: Root-level development commands
- {test-dir}/e2e/TEMPLATES.md: E2E test patterns
- scripts/create-bug.sh: Bug report generator
- scripts/precommit-tests.sh: Pre-commit test runner

Process Guidelines:
- Small changes: Direct implementation
- Big changes: Use coordinate-dev-workflow
- All code changes require tests
- All bugs must be documented

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

## Verification

After implementation, verify:

1. **CLAUDE.md exists** and contains all required sections
2. **Makefile works**: `make help` shows available commands
3. **Bug script works**: `./scripts/create-bug.sh` creates bug file
4. **Pre-commit tests work**: `./scripts/precommit-tests.sh` runs tests
5. **Directories created**: All required docs/ subdirectories exist
6. **Scripts executable**: `ls -la scripts/` shows execute permissions
7. **Tests pass**: Run `make test` to verify baseline

## Best Practices

1. **Explore First**: Always understand the existing codebase before making changes
2. **Adapt to Context**: Match testing frameworks and patterns to what exists
3. **Preserve Good Patterns**: Don't replace working processes
4. **Document Decisions**: Explain why certain requirements were chosen
5. **Make Scripts Portable**: Use shebangs and check for dependencies
6. **Use Conventional Commits**: Follow established commit message standards

## Related Skills

- **coordinate-dev-workflow**: Use for implementing big features
- **spec**: Create feature specifications
- **design**: Create technical designs
- **test**: Write comprehensive tests
