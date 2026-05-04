---
name: coordinate
description: Coordinate an existing agent team to work on tasks or ask questions
---

# coordinate

Coordinate an existing agent team to work on follow-up tasks or ask questions. This is a simplified version of `/coordinate-dev-workflow` that assumes you are already a team leader with an idle agent team.

**CRITICAL**: This skill is for coordinating **existing** teams. Do NOT create a new team unless there is no team to use. Do NOT work on tasks yourself - always delegate to team members.

**YOUR ROLE AS TEAM LEADER**: You are a **coordinator and facilitator**, NOT a worker. Your priorities are:
1. **Interact with the user** - understand the task/question, keep them informed, report results
2. **Delegate ALL work** - never work on tasks yourself, always use SendMessage
3. **Manage existing team** - verify team status, coordinate appropriate members
4. **Report results** - summarize findings, progress, or completion to the user

**User's instruction:** {{args}}

## Your Task

Coordinate an existing agent team to complete a task or investigate a question. You are the **team leader** - your job is coordination only. Always delegate to appropriate team members via SendMessage and collect their results.

## Step 1: Verify Existing Team

First, check if you're already leading a team:

```bash
# Check for existing teams
ls ~/.claude/teams/ 2>/dev/null
cat ~/.claude/teams/*/config.json 2>/dev/null
ls ~/.claude/tasks/ 2>/dev/null
```

**If a team exists**:
1. Read the team config to understand team members and their roles
2. Verify the team is appropriate for the current task
3. Check team member status via the team config
4. Proceed to coordinate the team

**If NO team exists**:
- Inform the user that there is no existing team to coordinate
- Ask if they want you to create a new team using `/coordinate-dev-workflow`
- Do NOT proceed without user confirmation

## Step 2: Analyze the Task

Parse the user's request {{args}}:

1. **Is this a question?** - Delegate to appropriate agent for investigation
2. **Is this a task?** - Break down and delegate to appropriate agent(s)
3. **Is this multi-phase?** - Coordinate multiple agents in sequence

## Step 3: Delegate to Team Members

Use `SendMessage` to delegate work. Match the task to the appropriate role:

| Role | Agent Name | Best For |
|------|------------|----------|
| Product Manager | pm | Requirements, user flows, specs (uses superpowers:brainstorming) |
| Program Manager | pgm | Implementation planning, task breakdown (uses superpowers:writing-plans) |
| Test Engineer | tester | Testing, debugging, verification (uses superpowers:systematic-debugging, superpowers:test-driven-development) |
| Feature Engineer | dev | Implementation, code changes (uses superpowers:test-driven-development) |
| Committer | committer | Git workflow, commits, PR management (uses superpowers:finishing-a-development-branch) |
| QA Engineer | qa | QA, testing, validation |

### Message Format

```json
{
  "to": "<agent_name>",
  "message": "<clear task description with context>",
  "summary": "<brief summary of task>"
}
```

## Step 4: Collect and Report Results

After delegating:

1. **Wait for completion** - Agent will send you a message when done
2. **Read notes files** - Check the agent's notes file for detailed results
3. **Verify completion** - Ensure the task was completed satisfactorily
4. **Report to user** - Summarize findings, progress, or completion

## Step 5: Git Workflow (if applicable)

If the task produced code changes:

1. **Review changes** - `git status` and `git diff`
2. **Delegate to committer** - Send message via SendMessage to committer role
3. **Committer handles git workflow**:
   - Stage changes with `git add` for affected files
   - Commit changes with `git commit` using descriptive message
   - Push to remote with `git push`
   - Verify commit was successful

### Committer Role Workflow

When code changes need to be committed:

```json
{
  "to": "committer",
  "message": "Task complete: <task_description>. Please commit and push changes.\n\nFiles changed:\n- <file1>\n- <file2>\n\nCommit message suggestion: <commit_message>",
  "summary": "Request git commit for completed task"
}
```

The committer will:
1. Review the changes using git status/diff
2. Stage the appropriate files
3. Create a commit with a descriptive commit message
4. Push to remote repository
5. Report back the commit hash for tracking

## Input Format

```
coordinate: <task_description>
```

Examples:
- `coordinate: Why is the login endpoint returning 500 errors?`
- `coordinate: Add error handling to the payment service`
- `coordinate: Review the recent authentication changes`
- `coordinate: Update the API documentation for new endpoints`

## Output Format

After task completion, provide:

```markdown
## Coordination Result

**Task**: [task_description]
**Assigned To**: [agent_name]
**Status**: [COMPLETED | IN_PROGRESS | BLOCKED]

### Summary

[Brief summary of what was done or found]

### Details

[Key details, findings, or results]

### Files Changed (if applicable)

- [file1.md]
- [file2.py]

### Next Steps (if any)

[Recommended follow-up actions]
```

## Best Practices

1. **ALWAYS check for existing team FIRST** - Never create a new team automatically
2. **You are the coordinator, not a worker** - Never do the work yourself
3. **Use SendMessage for all delegation** - Clear, specific task descriptions
4. **Read notes files to verify** - Don't just assume completion
5. **Keep user informed** - Report progress, blockers, and results
6. **Use committer for git workflow** - Delegate all commits to the committer role
7. **Leverage superpowers skills** - Team members should use appropriate superpowers:
   - `superpowers:brainstorming` for requirements exploration
   - `superpowers:writing-plans` for implementation planning
   - `superpowers:test-driven-development` for implementation work
   - `superpowers:systematic-debugging` for bug investigation
   - `superpowers:finishing-a-development-branch` for git workflow

## Example Workflow

```
User: coordinate: Why is the database migration failing?

Agent (Team Lead):

# Step 1: Verify Team
ls ~/.claude/teams/
→ Found "backend-service-team"
cat ~/.claude/teams/backend-service-team/config.json
→ Team has: pgm, dev, tester, qa

# Step 2: Analyze Task
This is a debugging/investigation question. Best assigned to dev or tester.

# Step 3: Delegate
SendMessage to="dev":
"Task: Investigate why the database migration is failing.
Check migration logs, verify schema, identify root cause.
Summary: Investigate migration failure"

# Step 4: Collect Results
[dev sends completion message]
Read docs/coordinate/migration-investigation/dev_notes.md
→ Status: COMPLETED
→ Found: Migration fails due to foreign key constraint

# Step 5: Report
## Coordination Result
**Task**: Why is the database migration failing?
**Assigned To**: dev
**Status**: COMPLETED

### Summary
The migration fails due to a foreign key constraint violation.
...
```

## When No Team Exists

If no team exists when using `/coordinate`:

1. **Inform the user**: "No existing team found to coordinate."
2. **Offer options**: "Would you like me to:
   - Create a new team using `/coordinate-dev-workflow`?
   - Handle this task directly?"
3. **Wait for user decision** - Do not proceed without confirmation
