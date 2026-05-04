---
name: coordinate-qa-fix-iteration
description: Coordinate team to execute QA cases, identify bugs via TDD, fix issues, and iterate until QA passes
---

# coordinate-qa-fix-iteration

Orchestrate a QA-driven bug fixing workflow by coordinating a team to execute test cases, identify issues (bugs or QA plan problems), apply TDD fixes, and re-test until all cases pass. This skill acts as a team lead, managing the entire debug-fix-test cycle while delegating all work to team members.

**CRITICAL**: Always use TeamCreate to spawn the entire agent team. Never use the Agent() tool. All team members work in their own tmux windows, keeping the current window clean for coordination only.

**YOUR ROLE AS TEAM LEADER**: You are a **coordinator and facilitator**, NOT a worker. Your priorities are:
1. **Interact with the user** - keep them informed, request decisions, report progress
2. **Delegate ALL work** - never execute tests, fix bugs, or update plans yourself
3. **Fix team creation issues first** - if agent team creation fails, resolve it BEFORE proceeding
4. **Reuse existing teams** - ALWAYS reuse an existing team if one exists, never shut down teams

**CRITICAL PRINCIPLE**: Always be a leader and coordinator. If team coordination fails:
- STOP and fix the issue
- NEVER fall back to direct work unless user explicitly requests it
- Your value is orchestration, not implementation

**User's instruction:** {{args}}

## Your Task

Coordinate a team of specialist agents to fix QA failures through iterative TDD cycles. You are the **team leader** - your job is coordination, not execution. Always delegate to appropriate team members via SendMessage, verify their results, and track progress until the QA case passes.

## Team Setup (REQUIRED FIRST STEP)

**IMPORTANT**: ALWAYS check for and reuse an existing team before creating a new one. Teams are persistent and reusable.

```bash
# Check for existing teams FIRST
ls ~/.claude/teams/ 2>/dev/null
cat ~/.claude/teams/*/config.json 2>/dev/null
```

**If a team exists**: Reuse it, spawn any missing agents
**If no team exists**: Create a new one using TeamCreate

```json
{
  "team_name": "qa-fix-<case_name>-team",
  "description": "Fix QA failures for <case_name>",
  "agent_type": "general-purpose"
}
```

## Team Roles

| Role | Agent Name | Skills | Responsibilities |
|------|------------|--------|------------------|
| **QA Engineer** | qa | `/qa-run` | Execute QA cases, document results, verify fixes |
| **Test Engineer** | tester | `superpowers:systematic-debugging`, `superpowers:test-driven-development`, `/test` | Write reproduction tests (Red phase), verify fix |
| **Bug Fixer** | dev | `superpowers:systematic-debugging`, `superpowers:test-driven-development`, `/feature`, `/test` | Fix bugs, write code to pass tests (Green phase) |
| **Committer** | committer | `superpowers:finishing-a-development-branch` | Git workflow, commits, PR management |
| **QA Planner** | planner | `/qa-plan` | Update QA plans if issues found in test cases |
| **DevOps Engineer** | devops | `/azure-*` (deployment) | Deploy code changes to target environment |

## Workflow Phases

### Phase 0: Team Setup & Context Gathering (REQUIRED - DO THIS FIRST)

#### Step 1: Check for Existing Team
```bash
# Check for existing teams
ls ~/.claude/teams/ 2>/dev/null
cat ~/.claude/teams/*/config.json 2>/dev/null
ls ~/.claude/tasks/ 2>/dev/null
```

**If an existing team exists:**
1. **Reuse the team** - Teams are designed to be persistent and reusable
2. **Verify team members** - Send a quick message to verify responsiveness
3. **Spawn any missing agents** - Add additional roles if needed for current work
4. **Proceed with workflow** - Update team tracking document and continue

**If NO team exists:**
1. Create a new team using `TeamCreate`
2. Proceed to spawn team members

#### Step 2: Gather Context from Existing Logs

Read existing coordination logs under `@docs/coordinate/tc-aa-*/` to understand:
- Previous QA execution results
- Bugs already identified
- Fixes already attempted
- Current state of the system

```bash
# Find relevant execution logs
find docs/coordinate -name "*qa*" -type f
find docs/coordinate -name "*execution*" -type f
cat docs/coordinate/tc-aa-*/qa_notes.md
```

#### Step 3: Initialize Tracking

1. Create coordination directory: `docs/coordinate/qa-fix-<case_name>/`
2. Create tracking document: `docs/coordinate/qa-fix-<case_name>/coordinate.md`
3. Create initial tasks using `TaskCreate`

### Phase 1: Initial QA Execution

**Agent**: qa

1. Send message via `SendMessage` to `qa`:
   ```
   Task: Execute QA test case <case_name>
   Use /qa-run skill.
   Test case location: docs/qa/cases/<case_name>.md
   Environment: <production/staging/local>

   **CRITICAL QA REQUIREMENTS:**
   - Test the REAL deployed service (production/staging)
   - NEVER use mock clients (TestClient, unittest.mock)
   - ALWAYS use actual HTTP requests (curl, httpie, requests)
   - Report MUST clearly state environment (PRODUCTION/STAGING/LOCAL)
   - Record all evidence (screenshots, logs, network traces)
   ```

2. Wait for agent completion
3. Read `docs/coordinate/qa-fix-<case_name>/qa_notes.md` for status
4. **If ALL tests pass**: Go to Phase 6 (Completion)
5. **If tests fail**: Proceed to Phase 2 (Bug Identification)

### Phase 2: Bug Identification & Classification

**You (Team Lead)**: Analyze QA results to classify issues

**Read QA execution report** and categorize failures:

| Issue Type | Action | Next Phase |
|------------|--------|------------|
| **Code Bug** | Create bug report, send to dev | Phase 3 (TDD Fix) |
| **QA Plan Issue** | Update test case, re-execute | Phase 4 (QA Plan Fix) |
| **Environment Issue** | Document, request infrastructure fix | Phase 5 (Infrastructure) |
| **Test Flakiness** | Stabilize test, re-execute | Phase 4 (QA Plan Fix) |

**For each bug found, create a bug report:**

```
docs/coordinate/bugs/BUG-<id>-<name>.md
```

Bug report template:
```markdown
# BUG-<id>: <Bug Title>

**Reported By:** QA Test Execution (<case_name>)
**Date:** YYYY-MM-DD
**Severity:** CRITICAL/HIGH/MEDIUM/LOW
**Status:** OPEN
**Component:** <Component Name>

---

## Summary
<Brief description of the bug>

---

## Steps to Reproduce
1. Step one
2. Step two
3. Observe error

---

## Expected Behavior
<What should happen>

---

## Actual Behavior
<What actually happens>

---

## Impact
<Who is affected and how>

---

## Root Cause
<Analysis of why this occurs>

---

## Suggested Fix
<Recommended approach to fix>

---

## Files Likely Requiring Changes
- File path 1
- File path 2

---

## Test Verification
After fix, verify:
1. Verification step 1
2. Verification step 2

---

## Evidence
- Link to QA execution report
- Screenshots/logs
```

### Phase 3: TDD Bug Fix (Red-Green-Blue)

#### 3a: Red Phase - Write Reproduction Test

**Agent**: tester

1. Send message via `SendMessage` to `tester`:
   ```
   Task: Write reproduction test for bug <bug-id>
   First use superpowers:systematic-debugging to analyze the bug.
   Then use superpowers:test-driven-development to write the test following TDD methodology.
   Then use /test skill to write the actual test code.
   Bug report location: docs/coordinate/bugs/BUG-<id>-<name>.md

   Write a test that:
   1. Reproduces the bug described in the bug report
   2. FAILS initially (red phase)
   3. Will PASS once the bug is fixed
   ```

2. Wait for agent completion
3. Read `docs/coordinate/qa-fix-<case_name>/test_notes.md` for status
4. Verify: Test should fail initially (red)

#### 3b: Green Phase - Fix the Bug

**Agent**: dev

1. Send message via `SendMessage` to `dev`:
   ```
   Task: Fix bug <bug-id> to make tests pass
   First use superpowers:systematic-debugging to analyze the root cause.
   Then use superpowers:test-driven-development to guide the fix.
   Then use /feature skill to implement the fix.
   Bug report: docs/coordinate/bugs/BUG-<id>-<name>.md
   Test location: <test_files_from_tester>

   Implement the fix until all tests pass.
   Focus on minimal change to fix this specific bug.
   ```

2. If tests still failing, iterate between `tester` and `dev`
3. Read `docs/coordinate/qa-fix-<case_name>/feature_notes.md` for status
4. Verify: All reproduction tests passing
5. **Checkpoint**: Commit fix with `git commit` and push
   - Use semantic commit message: `fix(<component>): fix <bug-description>`
   - Push to remote: `git push`

#### 3c: Deploy Code Changes

**Agent**: devops (or fallback to Bash tool for local environments)

After bugs are fixed and committed, deploy the changes to the target environment.

1. Send message via `SendMessage` to `devops`:
   ```
   Task: Deploy code changes to <environment> environment
   Deploy the latest commits to <environment>.
   Environment: <production/staging/dev/local>
   Commit to deploy: <commit_hash or "latest">

   **Deployment Requirements:**
   - For local/dev: Restart local services, rebuild if needed
   - For staging: Deploy to staging environment
   - For production: Coordinate with user before deploying
   - Verify deployment health after deployment
   - Report deployment status and any issues
   ```

2. **Environment-Specific Deployment Commands:**

   For **local** environment:
   ```bash
   # Restart local development server
   pkill -f "python.*backend" || true
   cd backend && python -m uvicorn api.main:app --reload &
   cd frontend && npm run dev &
   ```

   For **staging** environment:
   ```bash
   # Deploy to Azure staging slot
   az webapp deployment source sync --name <app-name> --slot staging --resource-group <rg>
   # Or use deployment pipeline
   ```

   For **production** environment:
   ```bash
   # Deploy to Azure production
   az webapp deployment source sync --name <app-name> --resource-group <rg>
   # Or use deployment pipeline with approval
   ```

3. Wait for deployment completion
4. Verify deployment health (check service status, health endpoints)
5. Read deployment notes: `docs/coordinate/qa-fix-<case_name>/deployment_notes.md`
6. **If deployment fails**: Diagnose issue, retry deployment
7. **If deployment succeeds**: Proceed to Phase 3d (Re-Execute QA)

#### 3d: Re-Execute QA (Iteration Loop)

**Agent**: qa

After deployment, re-execute the full QA test case to verify all fixes.

1. Send message via `SendMessage` to `qa`:
   ```
   Task: Re-execute QA test case <case_name> after deployment
   Use /qa-run skill.
   Test case location: docs/qa/cases/<case_name>.md
   Environment: <environment>
   This is iteration <N> - verify all previous bugs are fixed.

   **CRITICAL QA REQUIREMENTS:**
   - Test the REAL deployed service (must match deployment target)
   - Record all evidence (screenshots, logs, network traces)
   - Report MUST clearly state environment tested
   ```

2. Wait for agent completion
3. Read updated QA notes
4. **Decision Point:**

   ```
   ┌─ ALL tests pass?
   │  ├─ YES → Go to Phase 6 (Generate QA Report & Complete)
   │  └─ NO → Return to Phase 2 (Bug Identification) with new findings
   ```

5. **If tests fail**: Document new bugs found, increment iteration counter, continue loop
6. **If tests pass**: Proceed to Phase 6

### Phase 4: QA Plan Fix

**Agent**: planner

Use when the issue is with the QA plan itself (e.g., wrong test data, unclear steps, environment mismatch).

1. Send message via `SendMessage` to `planner`:
   ```
   Task: Update QA plan for <case_name>
   Use /qa-plan skill to update the test case.
   Issue: <description of what's wrong with the plan>
   Current plan location: docs/qa/cases/<case_name>.md

   Update the plan to fix the identified issue.
   ```

2. Wait for agent completion
3. Review updated plan
4. **Checkpoint**: Commit updated plan
5. Return to Phase 1 (Re-execute QA)

### Phase 5: Infrastructure Issues

**You (Team Lead)**: Document and request infrastructure fixes

When issues are environment/infrastructure related:

1. Document the issue in coordination notes
2. Report to user with required infrastructure changes
3. **Wait for infrastructure fix** - do not proceed until fixed
4. Once fixed, return to Phase 1 (Re-execute QA)

### Phase 6: Generate QA Report & Complete

**IMPORTANT**: Team members remain available for future work. Teams are persistent and reusable.

#### 6a: Generate QA Report

**Agent**: qa (or you as coordinator)

Generate a comprehensive QA execution report summarizing the entire fix iteration.

1. Create QA report: `docs/coordinate/qa-fix-<case_name>/qa-report-<date>.md`

2. QA Report Template:
   ```markdown
   # QA Execution Report: <Case Name>

   **Date:** YYYY-MM-DD
   **Environment:** <production/staging/local>
   **Executor:** QA Fix Iteration Workflow
   **Test Suite:** <test_suite_name>

   ---

   ## Executive Summary

   **Overall Result:** ✅ PASS / ❌ FAIL

   **Iterations Required:** N
   **Total Bugs Fixed:** X
   **Total Time:** <duration>

   ---

   ## Acceptance Criteria Status

   | AC ID | Criteria | Status | Notes |
   |-------|----------|--------|-------|
   | AC-1 | <criteria> | ✅/❌ | |
   | AC-2 | <criteria> | ✅/❌ | |

   ---

   ## Bugs Fixed

   ### BUG-<id>: <Bug Name>
   - **Severity:** <severity>
   - **Status:** CLOSED
   - **Fix Commit:** <commit_hash>
   - **Description:** <brief description>

   ---

   ## Iteration History

   ### Iteration 1: [YYYY-MM-DD HH:MM]
   - QA Result: ❌ FAIL
   - Bugs Found: X
   - Action: Started TDD fix cycle

   ### Iteration N: [YYYY-MM-DD HH:MM]
   - QA Result: ✅ PASS
   - All bugs closed

   ---

   ## Test Evidence

   - Screenshots: <link to screenshots>
   - Logs: <link to logs>
   - Network traces: <link to traces>

   ---

   ## Deployment Information

   - **Deployed Commit:** <commit_hash>
   - **Deployment Time:** [YYYY-MM-DD HH:MM]
   - **Deployment Status:** ✅ Success

   ---

   ## Conclusion

   <Summary of results>

   ---

   **Report Generated:** YYYY-MM-DD HH:MM
   **Reporter:** QA Fix Iteration Workflow
   ```

3. Commit the QA report:
   ```bash
   git add docs/coordinate/qa-fix-<case_name>/qa-report-<date>.md
   git commit -m "docs(qa): add QA execution report for <case_name>"
   git push
   ```

#### 6b: Complete or Auto-Move to Next Case

1. **Verify ALL acceptance criteria pass**
2. **Close all open bug reports**
3. **Notify all team members of completion** via `SendMessage`:
   ```json
   {
     "to": "*",
     "message": "QA fix iteration complete - all tests passing. Team members are now idle and available for new tasks."
   }
   ```

4. **Check for auto-move instruction**:
   - If `--next` option was provided with a list of cases, automatically start Phase 1 for the next case
   - If `--next` option was not provided, complete and report to user

5. **Final status update** in coordination document

6. **Summary report to user** with:
   - QA case status
   - Bugs fixed
   - Tests passing
   - QA report location
   - Team members are idle and ready for follow-up tasks
   - Next action (if --next was provided, indicate next case to process)

## Input Format

```
coordinate-qa-fix-iteration: <case_name> [options]
```

Options:
- `--environment=<env>`: Specify test environment (production/staging/local, default: staging)
- `--focus=<bug-id>`: Focus on fixing specific bug only
- `--start-at=<phase>`: Start at specific phase (e.g., `--start-at=fix`)
- `--next=<case1,case2,...>`: Auto-move to next QA cases after completion (comma-separated list)

Examples:
- `coordinate-qa-fix-iteration: tc-aa-004`
- `coordinate-qa-fix-iteration: tc-aa-001 --environment=staging`
- `coordinate-qa-fix-iteration: tc-aa-003 --focus=BUG-004`
- `coordinate-qa-fix-iteration: tc-aa-001 --next=tc-aa-002,tc-aa-003,tc-aa-004`
- `coordinate-qa-fix-iteration: tc-aa-001 --environment=production --next=tc-aa-002`

## Progress Tracking

Track all progress in `docs/coordinate/qa-fix-<case_name>/coordinate.md`:

### Main Tracking Document Template

```markdown
# QA Fix Coordination: <Case Name>

**Case**: <case_name>
**Started**: [YYYY-MM-DD HH:MM:SS]
**Status**: [IN_PROGRESS | COMPLETED | BLOCKED]
**Team**: [team_name]
**Environment**: [production/staging/local]

---

## Team Roster

| Role | Agent Name | Status | Notes |
|------|------------|--------|-------|
| QA Engineer | qa | [Active/Done] | |
| Test Engineer | tester | [Active/Done] | |
| Bug Fixer | dev | [Active/Done] | |
| Committer | committer | [Active/Done] | |
| QA Planner | planner | [Active/Done] | |

---

## Iteration History

### Iteration 1: [YYYY-MM-DD HH:MM]
- **QA Result**: ❌ FAIL - X bugs found
- **Bugs Identified**:
  - BUG-001: <name>
  - BUG-002: <name>
- **Action**: Starting TDD fix cycle

### Iteration 2: [YYYY-MM-DD HH:MM]
- **Bugs Fixed**:
  - ✅ BUG-001: <name> (commit: abc123)
- **Deployment**: Deployed to <environment> (commit: abc123)
- **QA Result**: ⚠️ PARTIAL - X bugs fixed, Y remain
- **Bugs Remaining**:
  - ❌ BUG-002: <name>
- **Action**: Continue fixing

### Iteration N: [YYYY-MM-DD HH:MM]
- **Bugs Fixed**:
  - ✅ BUG-002: <name> (commit: def456)
- **Deployment**: Deployed to <environment> (commit: def456)
- **QA Result**: ✅ PASS - All tests passing
- **All bugs closed**
- **QA Report**: docs/coordinate/qa-fix-<case_name>/qa-report-<date>.md
- **Action**: Workflow complete

---

## Bug Tracker

| Bug ID | Name | Severity | Status | Assigned To |
|--------|------|----------|--------|-------------|
| BUG-001 | <name> | HIGH | OPEN/CLOSED | dev |
| BUG-002 | <name> | MEDIUM | OPEN/CLOSED | dev |

---

## Acceptance Criteria Status

| AC ID | Criteria | Status | Notes |
|-------|----------|--------|-------|
| AC-1 | <criteria> | ✅/❌ | |
| AC-2 | <criteria> | ✅/❌ | |
```

## Communication Protocol

### Sending Tasks to Team Members

Use `SendMessage` with the agent's name:

```json
{
  "to": "qa",
  "message": "Task: Execute QA test case tc-aa-004. Use /qa-run skill. Focus on verifying BUG-004 fix.",
  "summary": "Assign QA execution to QA engineer"
}
```

### Receiving Updates from Team Members

- Team members will automatically send you messages when they complete work
- These messages appear as new conversation turns
- **Do NOT reply with text** - use SendMessage to coordinate
- Read their notes files to verify work quality

## Git Workflow (REQUIRED after EACH fix)

**CRITICAL**: The committer agent MUST commit and push changes after EACH bug fix. Do NOT proceed to next bug until changes are committed and pushed to git.

### After EACH completed bug fix:

1. **Verify the fix** - Read notes file, check test results
2. **Review changes** - Check what files were modified
3. **Notify committer** - Send message via SendMessage to committer
4. **Committer handles git workflow**:
   - Stage changes with `git add` for affected files
   - Commit changes with `git commit` using descriptive message
   - Push to remote with `git push` to share with team
   - Verify commit was successful
5. **Update tracking doc** - Mark bug as closed with commit hash
6. **Then proceed** - Only then re-execute QA

### Committer Role Workflow

After each bug fix completion, send message to `committer`:

```json
{
  "to": "committer",
  "message": "Bug fix complete: <BUG-ID>. Please commit and push changes.\n\nFiles changed:\n- <file1>\n- <file2>\n\nCommit message suggestion: fix(<component>): <bug-description>",
  "summary": "Request git commit for bug fix"
}
```

The committer will:
1. Review the changes using git status/diff
2. Stage the appropriate files
3. Create a commit with a semantic commit message
4. Push to remote repository
5. Report back the commit hash for tracking

### Commit Message Format

Use semantic commit messages:

```bash
# For bug fixes
git commit -m "fix(auth): store user object in localStorage after registration"

# For QA plan updates
git commit -m "test(qa): update tc-aa-004 with correct environment settings"

# For test additions
git commit -m "test(auth): add reproduction test for BUG-004 user not stored"
```

## Decision Tree: Bug vs QA Plan Issue

When QA fails, determine the root cause:

```
Is the bug in the code or the test?
├─ Code Bug (application behaves incorrectly)
│  ├─ Create bug report
│  ├─ Write reproduction test (Red)
│  ├─ Fix the bug (Green)
│  └─ Re-run QA to verify
│
├─ QA Plan Issue (test is wrong)
│  ├─ Test has incorrect steps/data
│  ├─ Test is unclear/ambiguous
│  ├─ Test environment mismatch
│  └─ Update QA plan and re-run
│
└─ Environment Issue (infrastructure problem)
   ├─ Configuration missing/wrong
   ├─ Service unavailable
   └─ Document and request fix
```

## When Team Coordination Fails

**IMPORTANT**: You are a **team leader and coordinator first**. Never fall back to direct work unless the user explicitly requests it.

If team creation/coordination fails:
1. **STOP immediately** - Do not proceed with the workflow
2. **Diagnose the issue** - Check logs, error messages, system state
3. **Fix the issue** - Resolve the team/infrastructure problem
4. **Retry team creation** - Only proceed once team is working
5. **Inform the user** - Report the issue and resolution

## Output Format

After each iteration and at completion, provide:

```markdown
## QA Fix Iteration Status Update

**Case**: <case_name>
**Iteration**: [N]
**Overall Progress**: [X bugs fixed / Y total bugs]

### Bug Status

| Bug ID | Name | Status | Notes |
|--------|------|--------|-------|
| BUG-001 | <name> | ✅ FIXED | |
| BUG-002 | <name | 🔧 IN PROGRESS | |
| BUG-003 | <name> | ❌ OPEN | |

### Recent Activity

- [Most recent agent activity with result]

### Next Steps

- [What happens next]

### Tracking Document

[docs/coordinate/qa-fix-<case_name>/coordinate.md]
```

## Best Practices

1. **ALWAYS reuse existing teams**: Before creating a new team, check if you're already leading one
2. **Fix team issues before proceeding**: If agent spawning or team creation fails, stop and fix it
3. **You are the coordinator, not a worker**: Prioritize user interaction over doing work
4. **Use SendMessage for all agent communication**: Never communicate via text
5. **Read existing logs first**: Learn from previous QA executions before starting
6. **Classify issues correctly**: Distinguish between code bugs, test issues, and infrastructure problems
7. **COMMIT AND PUSH AFTER EACH FIX**: This is REQUIRED, not optional
8. **Iterate until all tests pass**: Don't stop until QA case passes completely

## Example Workflow

```
User: coordinate-qa-fix-iteration: tc-aa-004 --environment=staging

Agent (Team Lead):

# Phase 0: Team Setup & Context Gathering
1. Check for existing team: `ls ~/.claude/teams/`
   - No existing team found, proceed to create new one

2. Gather context from existing logs:
   - Read docs/coordinate/tc-aa-execution/execution-report-2026-04-04.md
   - Read docs/coordinate/bugs/BUG-004-user-not-stored.md
   - Understand: Previous QA found user object not stored in localStorage

3. TeamCreate: "qa-fix-tc-aa-004-team"
   - Team created successfully

4. For each role (qa, tester, dev, planner, devops):
   - Agent(name=role, team_name="qa-fix-tc-aa-004-team")
   - Verify each agent responds to ping

5. Create tracking doc: docs/coordinate/qa-fix-tc-aa-004/coordinate.md

# === ITERATION 1 ===

# Phase 1: Initial QA Execution
6. SendMessage to="qa": "Execute tc-aa-004 QA test case against staging"
7. qa executes test, finds bugs
8. Read qa_notes.md → 2 bugs found
9. Create bug reports:
   - docs/coordinate/bugs/BUG-001-user-not-stored.md
   - docs/coordinate/bugs/BUG-002-settings-placeholder-data.md

# Phase 2: Bug Classification
10. Classify both bugs as Code Bugs → proceed to TDD fix

# Phase 3: TDD Bug Fix (Red-Green)
11. SendMessage to="tester": "Write reproduction test for BUG-001"
12. tester writes test → test fails (red) ✓
13. SendMessage to="dev": "Fix BUG-001 - store user object in localStorage"
14. dev implements fix
15. Verify tests pass (green) ✓
16. **COMMIT**: `git commit -m "fix(auth): store user object in localStorage after registration" && git push`

17. Repeat for BUG-002...

# Phase 3c: Deploy to Staging
18. SendMessage to="devops": "Deploy latest changes to staging environment"
19. devops deploys commit abc123 to staging
20. Verify deployment health: staging service is up
21. Read deployment_notes.md → Deployment successful

# Phase 3d: Re-Execute QA (Iteration 1 Verification)
22. SendMessage to="qa": "Re-execute tc-aa-004 against staging after deployment"
23. qa executes test
24. Read qa_notes.md → 1 bug remaining (BUG-002 not fully fixed)
25. Update coordinate.md: Iteration 1 partial success
26. Return to Phase 2 with new findings

# === ITERATION 2 ===

# Phase 2: Bug Classification (Round 2)
27. BUG-002 needs additional fix → proceed to TDD fix

# Phase 3: TDD Bug Fix (Round 2)
28. SendMessage to="dev": "Complete fix for BUG-002 - settings placeholder data"
29. dev implements remaining fix
30. **COMMIT**: `git commit -m "fix(settings): load real agency data instead of placeholder" && git push`

# Phase 3c: Deploy to Staging (Round 2)
31. SendMessage to="devops": "Deploy latest changes to staging environment"
32. devops deploys commit def456 to staging
33. Verify deployment health

# Phase 3d: Re-Execute QA (Iteration 2 Verification)
34. SendMessage to="qa": "Re-execute tc-aa-004 against staging - final verification"
35. qa executes test
36. Read qa_notes.md → ✅ ALL TESTS PASS
37. Update coordinate.md: Iteration 2 success

# Phase 6: Generate QA Report & Complete
38. Generate QA report: docs/coordinate/qa-fix-tc-aa-004/qa-report-2026-04-05.md
39. Commit QA report
40. Close all bug reports
41. SendMessage to="*": "QA fix iteration complete - all tests passing"
42. Final report to user:
   "tc-aa-004 now passing in staging environment.
    - 2 bugs fixed across 2 iterations
    - Commits: abc123, def456
    - QA Report: docs/coordinate/qa-fix-tc-aa-004/qa-report-2026-04-05.md
    Team members remain available"
```

### Example with Auto-Move to Next Case

```
User: coordinate-qa-fix-iteration: tc-aa-001 --environment=staging --next=tc-aa-002,tc-aa-003

[... tc-aa-001 workflow completes as above ...]

# Phase 6b: Auto-Move Detection
43. Check --next option: ["tc-aa-002", "tc-aa-003"]
44. Report to user: "tc-aa-001 complete. Starting tc-aa-002 automatically..."
45. Update coordinate.md: Case tc-aa-001 closed, starting tc-aa-002

# === AUTOMATICALLY START tc-aa-002 ===
46. Reuse existing team
47. Update tracking doc: docs/coordinate/qa-fix-tc-aa-002/coordinate.md
48. Begin Phase 1 for tc-aa-002...
[... continues with tc-aa-002 ...]

# After tc-aa-002 completes, automatically start tc-aa-003
# After tc-aa-003 completes, report final summary for all cases
```

## Continuing a Stopped Workflow

When continuing a workflow that was stopped:

1. **Check for existing team**: `ls ~/.claude/teams/` and `cat ~/.claude/teams/*/config.json`
2. **Reuse existing team**: If team exists → Reuse it; if no team exists → Create new one
3. **Verify team members**: Send ping message to each member, verify all respond
4. Read the existing tracking document: `docs/coordinate/qa-fix-<case_name>/coordinate.md`
5. Check which bugs are still open
6. Read the latest QA notes to understand current state
7. Resume from appropriate phase
8. Update the tracking document with continuation notes
