---
name: take-a-todo
description: Use when ready to work on an issue from the backlog - promotes the earliest (or specified) todo spec to docs/specs/, creates an isolated git worktree, and prepares for implementation
---

# take-a-todo

Promote a todo spec to active development with isolated git worktree. Moves spec from `docs/todo/` to `docs/specs/`, creates a worktree at `.worktree/{branch-name}/`, and loads context for planning.

**User's instruction**: {{args}}

## Your Task

1. List all specs in `docs/todo/`
2. If user specified a spec name, use that; otherwise take the earliest by date
3. Move the file: `docs/todo/{name}.md` → `docs/specs/{name}.md`
4. Create git worktree in `.worktree/{branch-name}/` for isolated development
5. Read the moved spec into context
6. Ask user what to do next

## When to Use

Ready to work on an issue from the backlog with isolated development environment.

## Steps

1. **List todo specs**:
   ```bash
   ls -la docs/todo/*.md 2>/dev/null | awk '{print $9}' | xargs -I{} basename {}
   ```

2. **Select spec**:
   - If user provided `{name}`, match partial string against filenames
   - If no argument, take earliest by date (oldest first)
   - Show which spec was selected

3. **Move to specs**:
   ```bash
   mv docs/todo/{filename} docs/specs/{filename}
   ```

4. **Create branch name from spec**:
   - Extract issue name from filename (remove date suffix)
   - Convert to branch-friendly format: `fix/{issue-name}` or `feat/{issue-name}`
   - Example: `registration-field-naming-2026-05-04.md` → `fix/registration-field-naming`

5. **Check current branch**:
   ```bash
   # Get current branch
   CURRENT_BRANCH=$(git branch --show-current)
   
   # Warn if on main/master
   if [[ "$CURRENT_BRANCH" == "main" || "$CURRENT_BRANCH" == "master" ]]; then
     echo "⚠️  WARNING: You are on $CURRENT_BRANCH branch"
     echo "Todo tasks should be worked on feature branches, not main/master"
     echo "Consider: git checkout -b feature/my-feature"
     echo ""
     read -p "Continue anyway? (y/N): " confirm
     if [[ "$confirm" != "y" && "$confirm" != "Y" ]]; then
       echo "Aborted. Please create a feature branch first."
       exit 1
     fi
   fi
   
   echo "Creating worktree from branch: $CURRENT_BRANCH"
   ```

6. **Create git worktree**:
   ```bash
   # Create worktree directory
   mkdir -p .worktree
   
   # Create new branch and worktree from current branch (NOT main)
   git worktree add -b {branch-name} .worktree/{branch-name} "$CURRENT_BRANCH"
   
   # Set session working directory to worktree
   cd .worktree/{branch-name}
   ```

7. **Load into context**: Read the moved spec file

8. **Prompt user**:
   ```
   What would you like to do with this spec?
   
   A) Review - Read through and provide feedback
   B) Revise - Make edits to the spec
   C) Plan - Start writing implementation plan (use superpowers:writing-plans)
   
   Worktree created: .worktree/{branch-name}/
   Working directory switched to worktree
   ```

## Input

Optional spec name (partial match allowed):
- `/take-a-todo` → Takes earliest todo spec
- `/take-a-todo registration` → Takes matching spec (e.g., `registration-field-naming-2026-05-04.md`)
- `/take-a-todo 2026-05-04` → Takes spec by date

## Output

- Spec moved to `docs/specs/`
- Git worktree created at `.worktree/{branch-name}/`
- New branch checked out: `{branch-name}`
- Spec content loaded in context
- User prompted for next action

## Branch Naming Convention

| Issue Type | Branch Prefix | Example |
|------------|---------------|---------|
| Bug fix | `fix/` | `fix/registration-field-naming` |
| Feature | `feat/` | `feat/user-authentication` |
| Refactor | `refactor/` | `refactor/api-structure` |
| Docs | `docs/` | `docs/api-guide` |
| Test | `test/` | `test/registration-flow` |

## Examples

### Example 1: Take earliest todo with worktree (from feature branch)

```
User: /take-a-todo

You: Found 2 todo specs:
  - registration-field-naming-2026-05-04.md (oldest)
  - demo-credentials-text-2026-05-04.md

Selected: registration-field-naming-2026-05-04.md

Current branch: feature/provision-update-flow

Moved: docs/todo/registration-field-naming-2026-05-04.md → docs/specs/

Branch name: fix/registration-field-naming

Creating worktree from: feature/provision-update-flow
git worktree add -b fix/registration-field-naming .worktree/fix/registration-field-naming feature/provision-update-flow

Worktree created: .worktree/fix/registration-field-naming/
Working directory: .worktree/fix/registration-field-naming
Branch: fix/registration-field-naming
Base branch: feature/provision-update-flow

[Loads spec content]

What would you like to do with this spec?
A) Review - Read through and provide feedback
B) Revise - Make edits to the spec
C) Plan - Start writing implementation plan (use superpowers:writing-plans)
```

### Example 2: Warning on main branch

```
User: /take-a-todo

You: Found 1 todo spec:
  - registration-field-naming-2026-05-04.md

Selected: registration-field-naming-2026-05-04.md

⚠️  WARNING: You are on main branch
Todo tasks should be worked on feature branches, not main/master
Consider: git checkout -b feature/my-feature

Continue anyway? (y/N): 

[User should create feature branch first]
```

### Example 2: Take specific spec

```
User: /take-a-todo registration

You: Found matching spec:
  - registration-field-naming-2026-05-04.md

Moved: docs/todo/registration-field-naming-2026-05-04.md → docs/specs/

Branch name: fix/registration-field-naming

Worktree created: .worktree/fix/registration-field-naming/

[Loads spec content]

Ready for review, revision, or planning.
```

### Example 3: No todos available

```
User: /take-a-todo

You: No todo specs found in docs/todo/

Tips:
- Use /issue-to-spec to create specs from QA findings
- Or specify an issue directly to create a new spec
```

## Directory Structure

```
docs/
├── todo/              # Issue backlog (newly discovered)
│   └── issue-name-2026-05-04.md
└── specs/             # Active specs (ready for implementation)
    └── promoted-issue-2026-05-04.md

.worktree/
└── fix/registration-field-naming/    # Isolated development environment
    ├── .git/                          # Worktree git link
    ├── backend/
    ├── frontend/
    └── docs/specs/
        └── registration-field-naming-2026-05-04.md

Git branches:
  main                                  # Protected base
  ├── feature/provision-update-flow     # Current feature branch (base for worktree)
  │   └── fix/registration-field-naming  # Worktree branch (created from feature branch)
```

## When Done: Merge Back

When implementation is verified and ready to ship:

```bash
# From worktree directory
cd .worktree/fix/registration-field-naming

# Ensure changes are committed
git status
git add .
git commit -m "fix: resolve registration field naming mismatch

- Change form field names from camelCase to snake_case  
- Add agency_name, confirm_password, accept_terms
- Update form submission handler

Fixes: registration-field-naming-2026-05-04.md"

# Switch back to main repo and base branch
cd ../..
git checkout {base-branch}  # The branch worktree was created from (e.g., feature/provision-update-flow)

# Merge worktree branch into base branch
git merge fix/registration-field-naming --no-ff

# Remove worktree
git worktree remove .worktree/fix/registration-field-naming

# Delete worktree branch after merge (optional)
git branch -d fix/registration-field-naming
```

**Note**: Merge into the same branch the worktree was created from (not necessarily main). Worktrees inherit from your current feature branch.

## Workflow Context

```
QA/Review → /issue-to-spec → docs/todo/ (committed)
                                   ↓
                         /take-a-todo (when ready)
                                   ↓
                    Move spec → docs/specs/
                    Create worktree → .worktree/{branch}/
                                   ↓
                    superpowers:writing-plans
                                   ↓
               superpowers:subagent-driven-development
                                   ↓
                    Verify implementation
                                   ↓
                         Merge worktree → main
                                   ↓
                    Remove .worktree/{branch}/
```

## Best Practices

1. **Feature branch first**: Always work from feature branches, not main/master
2. **Oldest first**: Default to taking the earliest spec (FIFO)
3. **Partial matching**: Accept substring matches for convenience
4. **Branch hygiene**: Use conventional branch names (fix/, feat/, etc.)
5. **Worktree isolation**: All work happens in worktree, base branch stays clean
6. **Clean merge**: Merge with --no-ff for clear history
7. **Cleanup**: Remove worktree after successful merge

## Safety Notes

**⚠️ Main Branch Warning**: 
- Worktrees should NOT be created from main/master branch
- Todo tasks belong on feature branches where the work is relevant
- If on main: You'll be prompted to create a feature branch first

## Integration with Other Skills

- **/issue-to-spec**: Creates todo specs → Use `/take-a-todo` when ready
- **superpowers:using-git-worktrees**: Reference for worktree best practices
- **superpowers:writing-plans**: After taking spec → Choose "Plan" to create implementation plan
- **superpowers:subagent-driven-development**: After planning → Execute with coordinated subagents
- **superpowers:finishing-a-development-branch**: When done, merge worktree back to main

## Worktree Commands Reference

```bash
# List all worktrees
git worktree list

# Remove worktree after merge
git worktree remove .worktree/{branch-name}

# Prune stale worktree references
git worktree prune

# Show worktree status
git worktree list --porcelain
```

## Technical Notes

- Worktrees share the same `.git` object database
- Commits in worktree are visible in all repos immediately
- Never delete worktree directory manually (use `git worktree remove`)
- Merge from base branch location, not from within worktree
- Worktree branch should merge back into the branch it was created from
