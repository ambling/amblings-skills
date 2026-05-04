---
name: take-a-todo
description: Use when ready to work on an issue from the backlog - promotes the earliest (or specified) todo spec to docs/specs/, creates an isolated git worktree, and prepares for implementation
---

# take-a-todo

Promote a todo spec to active development with isolated git worktree. Moves spec from `docs/todo/` to `docs/specs/`, delegates worktree creation to `using-git-worktrees`, and loads context for planning.

**User's instruction**: {{args}}

---

## Your Task

1. List all specs in `docs/todo/`
2. If user specified a spec name, use that; otherwise take the earliest by date
3. Move the file: `docs/todo/{name}.md` → `docs/specs/{name}.md`
4. Commit the move to git (in current branch)
5. **Delegate to `using-git-worktrees`** for worktree creation
6. Read the moved spec into context
7. Ask user what to do next

---

## When to Use

Ready to work on an issue from the backlog with isolated development environment.

---

## Steps

### 1. List todo specs
```bash
ls -la docs/todo/*.md 2>/dev/null | awk '{print $9}' | xargs -I{} basename {}
```

### 2. Select spec
- If user provided `{name}`, match partial string against filenames
- If no argument, take earliest by date (oldest first)
- Show which spec was selected

### 3. Move to specs
```bash
mv docs/todo/{filename} docs/specs/{filename}
```

### 4. Commit the move (in current branch before worktree)
```bash
git add docs/todo/ docs/specs/
git commit -m "docs(specs): promote {issue-name} from todo to active

Moved from docs/todo/ to docs/specs/ for implementation.

Spec: {filename}"
```

### 5. Create branch name from spec
- Extract issue name from filename (remove date suffix)
- Convert to branch-friendly format: `fix/{issue-name}` or `feat/{issue-name}`
- Example: `registration-field-naming-2026-05-04.md` → `fix/registration-field-naming`

### 6. Check current branch
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
```

### 7. Delegate to `using-git-worktrees`

Instead of manually creating worktrees, invoke the proper superpowers skill:

```
Now I'll use using-git-worktrees to create an isolated workspace for this branch.

Branch: {branch-name}
Base branch: {CURRENT_BRANCH}
```

The `using-git-worktrees` skill will:
- Select appropriate worktree directory (.worktrees, worktrees, or global location)
- Verify .gitignore safety
- Create the worktree with proper isolation
- Handle directory selection systematically

### 8. Load into context
Read the moved spec file

### 9. Prompt user
```
What would you like to do with this spec?

A) Review - Read through and provide feedback
B) Revise - Make edits to the spec
C) Plan - Start writing implementation plan (invokes writing-plans)

Worktree created by using-git-worktrees
Spec loaded: {filename}
```

---

## Input

Optional spec name (partial match allowed):
- `/take-a-todo` → Takes earliest todo spec
- `/take-a-todo registration` → Takes matching spec (e.g., `registration-field-naming-2026-05-04.md`)
- `/take-a-todo 2026-05-04` → Takes spec by date

---

## Output

- Spec moved to `docs/specs/`
- Git commit created for the move
- **Worktree created by `using-git-worktrees`**
- New branch checked out: `{branch-name}`
- Spec content loaded in context
- User prompted for next action

---

## Branch Naming Convention

| Issue Type | Branch Prefix | Example |
|------------|---------------|---------|
| Bug fix | `fix/` | `fix/registration-field-naming` |
| Feature | `feat/` | `feat/user-authentication` |
| Refactor | `refactor/` | `refactor/api-structure` |
| Docs | `docs/` | `docs/api-guide` |
| Test | `test/` | `test/registration-flow` |

---

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

Committed move to feature/provision-update-flow:
✓ git add docs/todo/ docs/specs/
✓ git commit -m "docs(specs): promote registration-field-naming from todo to active"

Branch name: fix/registration-field-naming

Now I'll use using-git-worktrees to create an isolated workspace.

[Invokes using-git-worktrees skill]

Worktree created: .worktrees/fix/registration-field-naming/
Working directory: .worktrees/fix/registration-field-naming
Branch: fix/registration-field-naming
Base branch: feature/provision-update-flow

[Loads spec content]

What would you like to do with this spec?
A) Review - Read through and provide feedback
B) Revise - Make edits to the spec
C) Plan - Start writing implementation plan (invokes writing-plans)
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

### Example 3: Take specific spec

```
User: /take-a-todo registration

You: Found matching spec:
  - registration-field-naming-2026-05-04.md

Current branch: feature/provision-update-flow

Moved: docs/todo/registration-field-naming-2026-05-04.md → docs/specs/

Committed move to feature/provision-update-flow
✓ git add docs/todo/ docs/specs/
✓ git commit -m "docs(specs): promote registration-field-naming from todo to active"

Branch name: fix/registration-field-naming

Now I'll use using-git-worktrees to create an isolated workspace.

[Invokes using-git-worktrees skill]

Worktree created: .worktrees/fix/registration-field-naming/
[Loads spec content]

Ready for review, revision, or planning.
```

### Example 4: No todos available

```
User: /take-a-todo

You: No todo specs found in docs/todo/

Tips:
- Use /issue-to-spec to create specs from QA findings
- Or specify an issue directly to create a new spec
```

---

## Directory Structure

```
docs/
├── todo/              # Issue backlog (newly discovered)
│   └── issue-name-2026-05-04.md
└── specs/             # Active specs (ready for implementation)
    └── promoted-issue-2026-05-04.md

.worktrees/            # Managed by using-git-worktrees
└── fix/registration-field-naming/    # Isolated development environment
    ├── .git/                          # Worktree git link
    ├── backend/
    ├── frontend/
    └── docs/specs/
        └── registration-field-naming-2026-05-04.md

Git branches:
  main                                  # Protected base
  ├── feature/provision-update-flow     # Current feature branch (base for worktree)
  │   └── fix/registration-field-naming  # Worktree branch (created by using-git-worktrees)
```

---

## When Done: Merge Back

When implementation is verified and ready to ship, use `finishing-a-development-branch`:

```
User: I'm done with this implementation.

You: I'll use finishing-a-development-branch to complete this work.

[Invokes finishing-a-development-branch skill]

The skill will:
1. Verify tests pass
2. Present merge options (local merge, PR, keep, discard)
3. Execute your choice
4. Cleanup worktree automatically
```

**Benefits of using finishing-a-development-branch:**
- Consistent completion workflow across all projects
- Proper worktree cleanup
- Safety checks before merge
- Handles both local and PR workflows

---

## Workflow Context

```
QA/Review → /issue-to-spec → docs/todo/ (committed)
                                   ↓
                         /take-a-todo (when ready)
                                   ↓
                    Move spec → docs/specs/
                    Commit move
                                   ↓
              using-git-worktrees
              (creates isolated workspace)
                                   ↓
                    writing-plans
              (from worktree, creates plan.md)
                                   ↓
      subagent-driven-development
              (executes implementation plan)
                                   ↓
                    Verify implementation
                                   ↓
      finishing-a-development-branch
              (handles merge, cleanup)
                                   ↓
                    Back to base branch
```

---

## Best Practices

1. **Feature branch first**: Always work from feature branches, not main/master
2. **Oldest first**: Default to taking the earliest spec (FIFO)
3. **Partial matching**: Accept substring matches for convenience
4. **Branch hygiene**: Use conventional branch names (fix/, feat/, etc.)
5. **Delegate worktree creation**: Let `using-git-worktrees` handle isolation
6. **Delegate completion**: Let `finishing-a-development-branch` handle merge/cleanup
7. **Trust superpowers**: Each skill has a focused responsibility

---

## Safety Notes

**⚠️ Main Branch Warning**: 
- Worktrees should NOT be created from main/master branch
- Todo tasks belong on feature branches where the work is relevant
- If on main: You'll be prompted to create a feature branch first

---

## Integration with Superpowers Skills

| Superpower Skill | Role | When Invoked |
|-----------------|------|--------------|
| `using-git-worktrees` | Creates isolated workspace | After committing spec move |
| `writing-plans` | Creates implementation plan.md | User chooses "Plan" option |
| `subagent-driven-development` | Executes implementation plan | After planning phase |
| `finishing-a-development-branch` | Handles merge and cleanup | When implementation is complete |
| `test-driven-development` | Writes tests before code | During implementation phase |
| `systematic-debugging` | Debugs issues found | When bugs are encountered |

---

## Delegation Benefits

By delegating worktree creation and completion to superpowers skills:

1. **Consistency**: Same workflow across all projects
2. **Safety**: Proper .gitignore checks and cleanup
3. **Simplicity**: take-a-todo focuses on spec promotion
4. **Expertise**: Each superpowers skill is optimized for its task
5. **Maintainability**: Superpowers updates improve all skills automatically

---

## Technical Notes

- take-a-todo creates the git commit for spec promotion
- Worktree creation is fully delegated to `using-git-worktrees`
- Worktree cleanup is handled by `finishing-a-development-branch`
- The skill acts as a coordinator: prepare → delegate → proceed

---

## Version

**Current:** 1.1.0
**Last Updated:** 2026-05-04
**Changes:** Refactored to delegate worktree/completion to superpowers skills
