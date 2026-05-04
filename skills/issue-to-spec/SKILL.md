---
name: issue-to-spec
description: Use when issues are found during QA or code review - converts findings into actionable specs in docs/todo/ and commits them
---

# issue-to-spec

Convert QA findings and code review issues into actionable todo specs with automatic git commit.

**User's instruction**: {{args}}

## Your Task

Record issues discovered during QA or code review as spec files in `docs/todo/` and commit them to git.

## When to Use

After QA testing or code review reveals issues that need to be tracked and fixed.

## Steps

1. **Check directory**: Ensure `docs/todo/` exists, create if needed
2. **Parse issues**: Extract issues from context (QA report, review feedback, etc.)
3. **Create specs**: For each issue:
   - Create file: `docs/todo/{issue-name}-{YYYY-MM-DD}.md`
   - Use brainstorming format as template
   - Include: severity, location, observed behavior, expected behavior, fix recommendations
4. **Commit to git**:
   ```bash
   git add docs/todo/
   git commit -m "docs(todo): add issue specs from QA/review

   - {issue-1-summary}
   - {issue-2-summary}

   Co-Authored-By: Claude Code <noreply@anthropic.com>"
   ```
5. **List specs**: Show all created files

## Input

Issues from:
- QA report (e.g., `/qa-plus` output)
- Code review feedback
- Manual issue identification

## Output

Committed `.md` files in `docs/todo/`

## Example Filenames

- `docs/todo/registration-field-naming-2026-05-04.md`
- `docs/todo/api-validation-errors-2026-05-04.md`
- `docs/todo/console-error-fix-2026-05-04.md`

## Spec Template

```markdown
# {Issue Title}

**Date**: {YYYY-MM-DD}
**Source**: {QA Report / Code Review / Manual}
**Priority**: {CRITICAL / HIGH / MEDIUM / LOW}
**Status**: {NEW / IN_PROGRESS / VERIFIED / FIXED}

---

## Problem Statement

{Brief description of the issue}

---

## Issue Details

**Severity**: {CRITICAL / HIGH / MEDIUM / LOW}
**Location**: {File path, component, or API endpoint}

**Observed Behavior**:
{What is happening}

**Expected Behavior**:
{What should happen}

**Evidence**:
{Code snippets, error messages, screenshots}

---

## Root Cause

{Analysis of why the issue occurs}

---

## Fix Recommendations

**Files to Modify**:
- `path/to/file1.ts`
- `path/to/file2.ts`

**Proposed Solution**:
{Description of the fix}

**Acceptance Criteria**:
1. [ ] {Criteria 1}
2. [ ] {Criteria 2}

---

## Testing Plan

{How to verify the fix works}

---

**Created By**: {source}
**Assigned To**: TBD
**Due Date**: {if applicable}
```

## Examples

### Example 1: Recording QA issues

```
User: /issue-to-spec

You: Found 2 issues from QA report:
- Registration field naming mismatch (CRITICAL)
- Misleading demo credentials text (LOW)

Created specs:
- docs/todo/registration-field-naming-2026-05-04.md
- docs/todo/demo-credentials-text-2026-05-04.md

Committed: docs(todo): add issue specs from QA/review
```

## Workflow Integration

```
QA/Review → /issue-to-spec → docs/todo/ (committed)
                                   ↓
                         /take-a-todo (when ready)
                                   ↓
                               docs/specs/
                                   ↓
                    superpowers:writing-plans
                                   ↓
               superpowers:subagent-driven-development
```

## Best Practices

1. **Date your specs**: Always include `{YYYY-MM-DD}` in filename
2. **One issue per file**: Keep specs focused and actionable
3. **Prioritize clearly**: Mark CRITICAL issues that block deployment
4. **Link evidence**: Reference screenshots, error logs, code locations
5. **Commit immediately**: Auto-commit ensures no issues are lost

## Integration with Other Skills

- **/qa-plus**: Generates QA reports → Use `/issue-to-spec` to create specs
- **/review**: Code review findings → Use `/issue-to-spec` to track issues
- **/take-a-todo**: Promote todo specs to active development
- **superpowers:writing-plans**: After `/take-a-todo` → Create implementation plan from spec
- **superpowers:subagent-driven-development**: After planning → Execute with coordinated subagents
