---
name: coordinate-dev-workflow
description: Spawn and lead a software engineering agent team to build features through a complete SDLC workflow
---

# coordinate-dev-workflow

Orchestrate a complete software development workflow by spawning and coordinating a team of specialist agents. This skill acts as a team lead, managing the entire lifecycle from specification to QA, while delegating all actual work to team members.

**CRITICAL**: Always use TeamCreate to spawn the entire agent team. Never use the Agent() tool. All team members work in their own tmux windows, keeping the current window clean for coordination only.

**YOUR ROLE AS TEAM LEADER**: You are a **coordinator and facilitator**, NOT a worker. Your priorities are:
1. **Interact with the user** - keep them informed, request decisions, report progress
2. **Delegate ALL work** - never implement, test, or design yourself
3. **Fix team creation issues first** - if agent team creation fails, resolve it BEFORE proceeding
4. **Reuse existing teams** - ALWAYS reuse an existing team if one exists, never shut down teams

**CRITICAL PRINCIPLE**: Always be a leader and coordinator. If team coordination fails:
- STOP and fix the issue
- NEVER fall back to direct work unless user explicitly requests it
- Your value is orchestration, not implementation

**IMPORTANT**: If TeamCreate or Agent tool fails with team-related errors:
- Stop and check for stale state in `~/.claude/teams/` and `~/.claude/tasks/`
- Clean up manually if needed before retrying
- The Agent tool requires a valid team context via `team_name` parameter

**User's instruction:** {{args}}

## Your Task

Coordinate a team of specialist agents to deliver a service/module/feature through a complete TDD-based development workflow. You are the **team leader** - your job is coordination, not execution. Always delegate to appropriate team members via SendMessage, verify their results via task notes, and track progress.

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
  "team_name": "<feature_name>-team",
  "description": "Develop <feature description>",
  "agent_type": "general-purpose"
}
```

## Team Roles

| Role | Agent Name | Skills | Responsibilities |
|------|------------|--------|------------------|
| **Product Manager** | pm | `superpowers:brainstorming`, `/spec` | Define requirements, user flows, functional specs |
| **Program Manager** | pgm | `superpowers:writing-plans` | Implementation planning, task breakdown |
| **Test Engineer** | tester | `superpowers:test-driven-development`, `/test` | Unit tests (TDD), integration tests, E2E tests |
| **Feature Engineer** | dev | `superpowers:test-driven-development`, `/feature` | Implementation to pass tests |
| **Committer** | committer | `superpowers:finishing-a-development-branch` | Git workflow, commits, PR management |
| **QA Engineer** | qa | `/qa-plan`, `/qa-run` | QA case generation and execution |

**Spawn each agent with their own tmux window**:

```bash
# Spawn agent in named tmux window
tmux new-window -n <role> -c "#{pane_current_path}"
# Then spawn the agent in that window
```

## Workflow Phases

### Team Reuse Decision Tree

Before starting any workflow, always check for an existing team:

```
Is there an existing team?
├─ No → Create new team and proceed
└─ Yes → ALWAYS reuse the existing team
    └─ Spawn any missing agents needed for current work
```

**IMPORTANT**: Always reuse existing teams. Teams are designed to be persistent and reusable across multiple features/tasks. Only create a new team if none exists.

### Phase 0: Team Setup (REQUIRED - DO THIS FIRST)

**CRITICAL**: Complete this phase BEFORE any other work. ALWAYS reuse existing teams.

#### Step 1: Check for Existing Team

First, check if you're already leading a team:

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

#### Step 2: Create the Team (if needed)

If no team exists, create a new one:

```json
{
  "team_name": "<feature_name>-team",
  "description": "Develop <feature description>",
  "agent_type": "general-purpose"
}
```

Use `TeamCreate` with these parameters.

#### Step 3: Spawn Team Members

For each role, spawn a teammate using the `Agent` tool:
- Use `team_name` parameter to join the team
- Use `name` parameter for the agent's name
- Each agent works independently

#### Step 4: Verify Team Creation

After spawning agents:
1. Verify all agents are responsive via `SendMessage`
2. Check team config at `~/.claude/teams/<team_name>/config.json`
3. If any agent fails to spawn or respond:
   - **STOP here** - do not proceed with workflow
   - Diagnose the issue (check logs, error messages)
   - Fix the issue (respawn agent, recreate team)
   - Only proceed once ALL team members are confirmed working

#### Step 5: Initialize Tracking

1. Create initial tasks using `TaskCreate`
2. Update team roster in tracking document
3. Document the workflow start

### Phase 1: Specification

**Agent**: pm (Product Manager)

1. Send message via `SendMessage` to `pm`:
   ```
   Task: Create specification for <feature>
   Use superpowers:brainstorming to explore user intent, requirements and design.
   Then use /spec skill to generate the specification document.
   Feature: <feature_name>
   Description: <description>

   Note: Both brainstorming and spec outputs will use date in directory name (YYYY-MM-DD_<feature> format).
   ```

2. Wait for agent completion (idle notification)
3. Read `docs/coordinate/<feature>/spec_notes.md` for status
4. **Checkpoint**: Present spec to user for review/questions
5. **AFTER APPROVAL**: Commit spec with `git commit` and push
6. **Skip if**: User specifies `--skip-confirm` or `--auto`

### Phase 2: Implementation Planning

**Agent**: pgm (Program Manager)

1. Send message via `SendMessage` to `pgm`:
   ```
   Task: Create implementation plan for <feature>
   Use superpowers:writing-plans based on approved design.
   Design location: docs/specs/YYYY-MM-DD_<feature>/design.md

   Note: The directory name includes date (YYYY-MM-DD_<feature> format).

   Create a detailed implementation plan with:
   - Step-by-step phases with clear tasks
   - Acceptance criteria for each phase
   - Testing strategy (unit, integration, E2E)
   - Dependencies and risks
   ```

2. Wait for agent completion
3. Read `docs/coordinate/<feature>/writing_plans_notes.md` for status
4. **Checkpoint**: Present plan to user for confirmation
5. **AFTER APPROVAL**: Commit plan with `git commit` and push
6. Allow iterations based on feedback
7. **Skip if**: User specifies `--skip-confirm` or `--auto`

### Phase 4: Test-Driven Development (Red-Green)

#### 4a: Red Phase - Unit Tests

**Agent**: tester

1. Send message via `SendMessage` to `tester`:
   ```
   Task: Write unit tests for <feature>
   Use superpowers:test-driven-development to follow TDD methodology.
   Then use /test skill to write the tests.
   Plan location: docs/implementation/<feature>/implementation_plan.md
   Ensure tests fail initially (red phase).
   ```

2. Wait for agent completion
3. Read `docs/coordinate/<feature>/test_notes.md` for status
4. Verify: All tests should fail initially (red)

#### 4b: Green Phase - Feature Implementation

**Agent**: dev

1. Send message via `SendMessage` to `dev`:
   ```
   Task: Implement <feature> to pass tests
   Use superpowers:test-driven-development to guide implementation.
   Then use /feature skill.
   Test location: <test_files_from_tester>
   Implement until all tests pass.
   ```

2. If tests still failing, send message to `tester` to update tests
3. Iterate via SendMessage between `tester` and `dev` until tests pass
4. Read `docs/coordinate/<feature>/feature_notes.md` for status
5. Verify: All unit tests passing
6. **Checkpoint**: Commit code with `git commit` and push
   - Use semantic commit message: `feat(<feature>): implement <feature>`
   - Push to remote: `git push`

### Phase 5: Integration & E2E Testing

**Agent**: tester

1. Send message via `SendMessage` to `tester`:
   ```
   Task: Write integration and E2E tests for <feature>
   Use /test skill for integration testing.
   Verify all tests pass.
   ```

2. Wait for agent completion
3. Read `docs/coordinate/<feature>/test_notes.md` for status
4. Verify: All tests pass
5. **Checkpoint**: Commit test files with `git commit` and push
   - Use semantic commit message: `test(<feature>): add integration and E2E tests`
   - Push to remote: `git push`

### Phase 6: QA Planning

**Agent**: qa

1. Send message via `SendMessage` to `qa`:
   ```
   Task: Generate QA test cases for <feature>
   Use /qa-plan skill.
   Feature location: docs/specs/<feature>.md
   ```

2. Wait for agent completion
3. Read `docs/coordinate/<feature>/qa_plan_notes.md` for status
4. **Checkpoint**: Present QA cases to user for review
5. **AFTER APPROVAL**: Commit QA plan with `git commit` and push
6. **Skip if**: User specifies `--skip-confirm` or `--auto`

### Phase 7: QA Execution

**Agent**: qa

1. Send message via `SendMessage` to `qa`:
   ```
   Task: Execute QA tests for <feature>
   Use /qa-run skill.
   Test cases location: docs/qa/cases/<feature>/
   Log results to docs/qa/log/<feature>/

   **CRITICAL QA REQUIREMENTS:**
   - QA must test the REAL deployed service (production/staging)
   - NEVER use mock clients (TestClient, unittest.mock)
   - ALWAYS use actual HTTP requests (curl, httpie, requests)
   - Report MUST clearly state environment (PRODUCTION/STAGING/LOCAL)
   - QA ≠ Unit Testing - QA verifies actual deployment
   ```

2. Wait for agent completion
3. Read `docs/coordinate/<feature>/qa_run_notes.md` for status
4. **VERIFY**: Check that QA report clearly states the environment tested
5. Track results and issues found
6. **Checkpoint**: Record QA results

### Phase 8: Completion & Summary

**IMPORTANT**: Team members remain available for future work. Teams are persistent and reusable.

1. **Notify all team members of completion** via `SendMessage`:
   ```json
   {
     "to": "*",
     "message": "Workflow complete - all phases finished successfully. Team members are now idle and available for new tasks."
   }
   ```

2. **Final status update** in coordination document

3. **Summary report to user** with:
   - Feature delivered
   - All phases completed
   - Team members are idle and ready for follow-up tasks

**NOTE**: Team members remain available for `/coordinate` to delegate follow-up tasks. Use `/coordinate` for quick follow-up questions or tasks without creating a new team.

## Input Format

```
coordinate-dev-workflow: <feature_name> <description> [options]
```

Options:
- `--skip-confirm` or `--auto`: Skip user confirmation checkpoints
- `--start-at=<phase>`: Start at a specific phase (e.g., `--start-at=design`)
- `--focus=<role>`: Focus on specific role's work

Examples:
- `coordinate-dev-workflow: user-service "User authentication with JWT"`
- `coordinate-dev-workflow: payment-gateway "Stripe integration" --auto`
- `coordinate-dev-workflow: api-client "REST client library" --start-at=design`

## Progress Tracking

Track all progress in `docs/coordinate/<feature_name>.md` and read individual skill notes:

**Important**: Each specialist agent skill updates its own notes file. As the coordinator:

1. **Read skill notes** after each agent completes work
2. **Parse the status** to understand what was done
3. **Update main tracking doc** with summary
4. **Proceed to next phase** based on status
5. **Use TaskCreate/TaskUpdate/TaskList** to track tasks

### Skill Notes Files (each skill writes its own)

| Skill | Notes File | Status Field |
|-------|------------|--------------|
| spec | `docs/coordinate/<feature>/spec_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| superpowers:writing-plans | `docs/coordinate/<feature>/writing_plans_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| test | `docs/coordinate/<feature>/test_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| feature | `docs/coordinate/<feature>/feature_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| qa-plan | `docs/coordinate/<feature>/qa_plan_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| qa-run | `docs/coordinate/<feature>/qa_run_notes.md` | Status: COMPLETED/IN_PROGRESS/BLOCKED |
| report | `docs/coordinate/<feature>/report_notes.md` | Status: COMPLETED |

### Task Tracking

Use TaskCreate to create tasks for each phase:

```
TaskCreate: "Create spec for <feature>"
TaskCreate: "Design <feature> system"
TaskCreate: "Plan <feature> implementation"
...
```

Assign tasks to team members via SendMessage with the task details.

### Reading Agent Status

After sending a task via SendMessage, wait for the agent to complete:
1. Agent will send you a message when done
2. Read their notes file to verify completion
3. Check the **Status** field at the top of the notes file

### Main Tracking Document Template

```markdown
# Development Coordination: <Feature Name>

**Feature**: [feature_name]
**Started**: [YYYY-MM-DD HH:MM:SS]
**Status**: [IN_PROGRESS | COMPLETED | BLOCKED]
**Team**: [team_name]

---

## Team Roster

| Role | Agent Name | Tmux Window | Status | Notes |
|------|------------|-------------|--------|-------|
| Product Manager | pm | pm | [Active/Done] | |
| Program Manager | pgm | pgm | [Active/Done] | |
| Test Engineer | tester | tester | [Active/Done] | |
| Feature Engineer | dev | dev | [Active/Done] | |
| Committer | committer | committer | [Active/Done] | |
| QA Engineer | qa | qa | [Active/Done] | |

---

## Workflow Progress

### Phase 1: Specification
- **Status**: [COMPLETED | IN_PROGRESS | PENDING]
- **Assigned To**: pm
- **Output**: [docs/specs/YYYY-MM-DD_<feature>/spec.md]
- **Notes File**: [docs/coordinate/<feature>/spec_notes.md]
- **User Review**: [Approved | Rejected | Pending]
- **Notes**: [Any issues or iterations]

### Phase 2: Implementation Planning
- **Status**: [COMPLETED | IN_PROGRESS | PENDING]
- **Assigned To**: pgm
- **Output**: [docs/implementation/<feature>/implementation_plan.md]
- **Notes File**: [docs/coordinate/<feature>/writing_plans_notes.md]
- **User Review**: [Approved | Rejected | Pending]
- **Notes**: [Any issues or iterations]

[... continue for all phases ...]
```

## Communication Protocol

### Sending Tasks to Team Members

Use `SendMessage` with the agent's name:

```json
{
  "to": "pm",
  "message": "Task: Create specification for user-auth feature. Use /spec skill. Description: JWT-based authentication with refresh tokens.",
  "summary": "Assign spec creation to PM"
}
```

### Receiving Updates from Team Members

- Team members will automatically send you messages when they complete work
- These messages appear as new conversation turns
- **Do NOT reply with text** - use SendMessage to coordinate
- Read their notes files to verify work quality

### To User

- Report phase completions
- Request confirmations at checkpoints
- Summarize agent results
- Highlight blockers needing decisions
- Provide periodic progress updates

### Coordinating Handoffs Between Roles

When one agent completes work and another needs to start:

```json
{
  "to": "pgm",
  "message": "Specification approved by user. Please proceed with implementation planning. Spec location: docs/specs/YYYY-MM-DD_user-auth/spec.md",
  "summary": "Assign planning task to Program Manager"
}
```

## Git Workflow (REQUIRED after EACH phase)

**CRITICAL**: The committer agent MUST commit and push changes after EACH completed phase. Do NOT proceed to the next phase until changes are committed and pushed to git.

### After EACH completed phase:

1. **Verify the agent's work** - Read notes file, check outputs
2. **Review changes** - Check what files were created/modified
3. **Notify committer** - Send message via SendMessage to committer
4. **Committer handles git workflow**:
   - Stage changes with `git add` for affected files
   - Commit changes with `git commit` using descriptive message
   - Push to remote with `git push` to share with team
   - Verify commit was successful
5. **Update tracking doc** - Mark phase as complete with commit hash
6. **Then proceed** - Only then assign next task

### Committer Role Workflow

After each phase completion, send message to `committer`:

```json
{
  "to": "committer",
  "message": "Phase complete: <Phase Name>. Please commit and push changes.\n\nFiles changed:\n- <file1>\n- <file2>\n\nCommit message suggestion: <commit_message>",
  "summary": "Request git commit for phase completion"
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
# For specification phase
git commit -m "docs(spec): add user authentication specification"

# For design phase
git commit -m "docs(design): add JWT auth architecture design"

# For implementation phase
git commit -m "feat(auth): implement JWT authentication with refresh tokens"

# For tests phase
git commit -m "test(auth): add unit and integration tests for auth endpoints"

# For QA phase
git commit -m "test(qa): add QA test cases for authentication flows"

# For fixes
git commit -m "fix(auth): resolve token expiration edge case"
```

### Push Protocol

After EVERY commit:

```bash
# Push to remote repository
git push

# For new branches, set upstream
git push -u origin <branch-name>
```

### Verification Protocol

After each agent completes work:

1. **Read the notes file** - Check the skill's notes file for status
2. **Verify quality** - Check against requirements and standards
3. **Review changes** - `git status` and `git diff` to see what changed
4. **Commit changes** - Use bash to commit the work
5. **Push to remote** - Use bash to push commits
6. **Update tracking doc** - Mark phase as complete with results
7. **Send next task** - Use SendMessage to assign next agent

## Handling Blockers & Iterations

When issues arise:

1. **Identify the blocking phase** - Which phase needs rework?
2. **Send message to appropriate agent** - Use SendMessage with the role's name
3. **Specify iteration requirements** - What needs to change?
4. **Verify iteration** - Read their updated notes file
5. **Update tracking doc** - Record iteration and reason
6. **Resume workflow** - Send message to continue from appropriate phase

Common iteration scenarios:
- **Spec changes** → Send message to `pm`, cascade to `pgm`
- **Test failures** → Send messages between `tester` and `dev`
- **QA failures** → Send message to `dev` for fixes

## When Team Coordination Fails

**IMPORTANT**: You are a **team leader and coordinator first**. Never fall back to direct work unless the user explicitly requests it.

If team creation/coordination fails:
1. **STOP immediately** - Do not proceed with the workflow
2. **Diagnose the issue** - Check logs, error messages, system state
3. **Fix the issue** - Resolve the team/infrastructure problem
4. **Retry team creation** - Only proceed once team is working
5. **Inform the user** - Report the issue and resolution

### Troubleshooting Steps

**TeamCreate fails with "Already leading team"**:
```bash
# Check what team exists
ls ~/.claude/teams/
cat ~/.claude/teams/*/config.json

# Reuse the existing team instead of creating a new one
```

**Agent tool fails with "Team does not exist"**:
```bash
# Verify team was created
cat ~/.claude/teams/<team-name>/config.json

# If team directory doesn't exist, create it with TeamCreate
```

**Agent not responding to SendMessage**:
- Wait longer (some agents take time to process)
- Check if agent spawned correctly
- Respawn the specific agent

### When to Involve the User

- If team creation fails after 3 retry attempts with different approaches
- If you encounter a bug or limitation in the TeamCreate/Agent tools
- If the workflow cannot proceed due to infrastructure issues

**Then** ask the user: "Team coordination is encountering issues. Would you like me to continue debugging the team infrastructure, or would you prefer I handle this task directly?"

### NEVER Do This

- ❌ Fall back to direct work silently
- ❌ Use Read/Edit/Write tools for implementation without user consent
- ❌ Skip team creation because it's "easier"
- ❌ Proceed with workflow when team is broken

### ALWAYS Do This

- ✅ Stop and fix team issues first
- ✅ Keep the user informed of infrastructure problems
- ✅ Be a coordinator, not a worker
- ✅ Only use direct work when user explicitly requests it

## Output Format

After completion (or at checkpoints), provide:

```markdown
## Workflow Status Update

**Feature**: [feature_name]
**Current Phase**: [Phase N - Phase Name]
**Overall Progress**: [X]%

### Phase Status

| Phase | Agent | Status | Output |
|-------|-------|--------|--------|
| Specification | pm | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| Implementation Planning | pgm | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| Unit Tests (Red) | tester | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| Implementation (Green) | dev | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| Integration Tests | tester | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| QA Planning | qa | [COMPLETED | IN_PROGRESS | PENDING] | [link] |
| QA Execution | qa | [COMPLETED | IN_PROGRESS | PENDING] | [link] |

### Recent Activity

- [Most recent agent activity with result]

### Next Steps

- [What happens next]

### Tracking Document

[docs/coordinate/<feature_name>.md]
```

## Best Practices

1. **ALWAYS reuse existing teams**: Before creating a new team, check if you're already leading one
   - If team exists: reuse it, spawn additional agents as needed
   - If no team exists: create a new one
   - Never delete/shutdown teams automatically

2. **Fix team issues before proceeding**: If agent spawning or team creation fails
   - STOP and diagnose the issue
   - Fix it before any workflow work begins
   - Only proceed when ALL team members are confirmed working

3. **You are the coordinator, not a worker**: Prioritize user interaction over doing work
   - Keep the user informed
   - Delegate ALL implementation work to team agents
   - Never write code, tests, or docs yourself

4. **Use SendMessage for all agent communication**: Never communicate via text
5. **Keep this window clean**: All agent work happens in their own contexts
6. **Read notes files**: Always verify work by reading the skill notes
7. **Use TaskCreate for tracking**: Track major tasks in the task list
8. **COMMIT AND PUSH AFTER EACH PHASE**: This is REQUIRED, not optional
   - Verify work → Commit → Push → Then proceed to next phase
   - Never accumulate uncommitted work across phases
   - Use semantic commit messages

## Example Workflow

```
User: coordinate-dev-workflow: user-auth "JWT-based authentication"

Agent (Team Lead):

# Phase 0: Team Setup
1. Check for existing team: `ls ~/.claude/teams/`
   - No existing team found, proceed to create new one

2. TeamCreate: "user-auth-team"
   - Team created successfully

3. For each role (pm, pgm, tester, dev, qa):
   - Agent(name=role, team_name="user-auth-team")
   - Verify each agent responds to ping

4. All agents confirmed working → Proceed with workflow

# Phase 1: Specification
5. SendMessage to="pm": "Create spec for JWT authentication. Use superpowers:brainstorming first to explore requirements and design."
6. pm uses superpowers:brainstorming to create design doc at docs/superpowers/specs/YYYY-MM-DD-jwt-auth-design.md
7. pm uses /spec to create docs/specs/YYYY-MM-DD_user-auth/spec.md
8. Read docs/coordinate/user-auth/spec_notes.md → Status: COMPLETED
9. Present spec to user → User approves
10. **COMMIT**: `git commit -m "docs(spec): add user authentication specification" && git push`

# Phase 2: Implementation Planning
11. SendMessage to="pgm": "Create implementation plan for JWT authentication. Use superpowers:writing-plans based on approved spec."
12. pgm creates implementation plan at docs/implementation/user-auth/implementation_plan.md
13. Read docs/coordinate/user-auth/writing_plans_notes.md → Status: COMPLETED
14. Present plan to user → User approves
15. **COMMIT**: `git commit -m "docs(plan): add JWT auth implementation plan" && git push`

# [Continue through remaining phases...]
# ... (implementation, testing, QA, etc.)

# Phase 7: Completion & Summary
55. All phases complete
56. SendMessage to="*": "Workflow complete - all phases finished successfully. Team members are now idle and available for new tasks."
57. Final report to user: "user-auth feature delivered. Team members remain available for follow-up tasks using /coordinate"

# Later - User requests another feature
User: coordinate-dev-workflow: payment "Stripe integration"

Agent (Team Lead):
# Phase 0: Team Setup
1. Check for existing team: `ls ~/.claude/teams/`
   - Found "user-auth-team" - REUSE IT

2. Verify existing team members are responsive
3. Spawn any missing agents if needed
4. Proceed with workflow - no new team created
```

## Continuing a Stopped Workflow

When continuing a workflow that was stopped:

1. **Check for existing team**:
   ```bash
   ls ~/.claude/teams/
   cat ~/.claude/teams/*/config.json
   ```

2. **Reuse existing team**:
   - If team exists → Reuse it (always)
   - If no team exists → Create new one

3. **Verify team members**:
   - Send ping message to each member
   - Verify all respond
   - Respawn any non-responsive members

4. Read the existing tracking document: `docs/coordinate/<feature>.md`
5. Check which phase was in progress
6. Read the last agent's notes file to understand current state
7. Resume from that phase by sending appropriate message
8. Update the tracking document with continuation notes

**Example continuation**:
```
1. Check ~/.claude/teams/ → Found "backend-service-qa-team"
2. Reuse the existing team
3. SendMessage to all members → Verify responsiveness
4. Read docs/coordinate/backend-service-qa.md
5. See Phase 4 (QA Execution) was IN PROGRESS
6. Read docs/coordinate/backend-service-qa/qa_run_notes.md
7. Send message to "qa": "Continue QA execution from where you stopped"
8. Monitor progress via incoming messages
9. When complete: Notify team workflow is done, team remains available
```
