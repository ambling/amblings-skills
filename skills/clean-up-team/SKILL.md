---
name: clean-up-team
description: Stop all running tasks, shutdown agents, kill tmux panes, and delete the current agent team
---

# clean-up-team

Clean up and delete the current agent team. This stops all running tasks, shuts down agent processes, kills their tmux panes, and removes the team configuration.

**User's instruction:** {{args}}

## Your Task

Cleanly shutdown the current agent team by:
1. Stopping any running tasks
2. Shutting down all team agents
3. Killing their tmux panes
4. Deleting the team configuration

## Approach

1. **Identify the current team** - Check for existing team in `~/.claude/teams/`
2. **List team members** - Read team config to get all agent names and tmux pane IDs
3. **Stop running tasks** - Use `TaskStop` on any in-progress tasks for this team
4. **Kill tmux panes** - For each agent with a tmux pane ID, kill that pane
5. **Delete team directory** - Remove the team directory from `~/.claude/teams/`
6. **Delete task directory** - Remove the corresponding task directory if it exists

## Input Format

No arguments required. Operates on the current active team.

## Output Format

```
## Team Cleanup Complete

**Team:** {team_name}
**Agents Stopped:** {count}
**Tmux Panes Killed:** {list}
**Team Directory Deleted:** {path}

Team has been removed and all resources cleaned up.
```

## Examples

```
User: /clean-up-team

Agent: # Step 1: Check for existing team
ls ~/.claude/teams/
→ Found: qa-fix-multi-role-support-team

# Step 2: Read team config
cat ~/.claude/teams/qa-fix-multi-role-support-team/config.json
→ Agents: dev (%79), qa (%81), tester (%82)

# Step 3: Stop any running tasks
TaskList → No in-progress tasks

# Step 4: Kill tmux panes
tmux kill-pane -t %79
tmux kill-pane -t %81
tmux kill-pane -t %82

# Step 5: Delete team directory
rm -rf ~/.claude/teams/qa-fix-multi-role-support-team
rm -rf ~/.claude/tasks/qa-fix-multi-role-support-team

## Team Cleanup Complete

**Team:** qa-fix-multi-role-support-team
**Agents Stopped:** 3
**Tmux Panes Killed:** %79, %81, %82
**Team Directory Deleted:** ~/.claude/teams/qa-fix-multi-role-support-team

Team has been removed and all resources cleaned up.
```

## Safety Checks

Before proceeding, verify:
- The team exists and is not actively needed
- All critical work has been completed or committed
- No other sessions depend on this team
