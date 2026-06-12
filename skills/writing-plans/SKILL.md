---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans (Custom)

This skill is based on the upstream superpowers `writing-plans` skill (see `sp-SKILL.md` for reference) with custom modifications.

**Upstream base:** `sp-SKILL.md` (symlink to `superpowers/skills/writing-plans/SKILL.md`)
**Customizations:** Task-oriented format, no code samples, acceptance criteria, E2E test coverage, QA case updates.

---

## Overview

Write implementation plans as clear, self-contained tasks. Each task defines what to build and how to verify it — no code samples, just precise intent and acceptance criteria. The plan must include an E2E test task that validates the full feature against its spec, and a QA task that ensures existing QA cases are updated and the feature works end-to-end.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

Each task is a self-contained unit of work with a clear definition and acceptance criteria. **Do not include code samples** — describe intent, inputs, outputs, and constraints precisely enough that an engineer can implement without ambiguity.

````markdown
### Task N: [Component Name]

**Definition:** [What this task builds — the precise requirement and scope]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py` ([what changes])
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test** — [Describe what the test covers: inputs, expected outputs, edge cases]
- [ ] **Step 2: Run test to verify it fails** — Run: `pytest tests/path/test.py::test_name -v`. Expected: FAIL with appropriate error.
- [ ] **Step 3: Implement** — [Describe the logic/approach — what to build, not how to code it. Specify constraints, invariants, error handling.]
- [ ] **Step 4: Run test to verify it passes** — Run: `pytest tests/path/test.py::test_name -v`. Expected: PASS.
- [ ] **Step 5: Commit** — `git commit -m "feat: [what was added]"`

**Acceptance Criteria:**
- [ ] [Measurable condition 1 — e.g., "Function returns X when given Y"]
- [ ] [Measurable condition 2 — e.g., "Error case returns status 422 with message"]
- [ ] [Measurable condition 3 — e.g., "All tests pass, no regressions"]
````

## No Placeholders

Every task must contain enough detail for implementation without ambiguity. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases" (be specific about which errors and how)
- "Write tests for the above" (without describing what the tests cover)
- "Similar to Task N" (repeat the description — the engineer may be reading tasks out of order)
- Steps that describe what to do without enough detail to act

## Build and Deploy to Local Dev

The plan **must** include a task that rebuilds the application and deploys to the local dev environment after all implementation tasks are complete. This ensures the running app reflects all changes before E2E tests and QA verification run.

```markdown
### Task N+1: Build & Deploy — Local Dev

**Definition:** Rebuild the application with all implemented changes and deploy to the local development environment. Verify the app starts cleanly and is accessible.

- [ ] **Step 1: Build** — Run the build command for the project (e.g., `npm run build`, `docker build`). Ensure build completes with no errors.
- [ ] **Step 2: Deploy to local** — Deploy to local dev environment (e.g., `docker compose up`, `npm run dev`, local k8s). Use the same process the project uses for local development.
- [ ] **Step 3: Smoke test** — Open the app in browser or hit the health endpoint. Confirm it loads without errors. Check console for unexpected warnings.
- [ ] **Step 4: Commit build config changes** (if any) — `git commit -m "chore: update build config for [feature]"`

**Acceptance Criteria:**
- [ ] Build completes successfully with no errors
- [ ] App is running and accessible at the local dev URL
- [ ] No startup errors or crashes in logs
- [ ] Existing features still work (no regressions from deploy)
```

## E2E Test Coverage

The plan **must** include a task that creates E2E tests covering the feature spec end-to-end. This task validates the complete user-facing behavior described in the spec.

```markdown
### Task N+2: E2E Test — [Feature Name]

**Definition:** End-to-end tests that validate the full feature flow against the spec. Tests must cover the happy path and key error scenarios outlined in the spec.

**Files:**
- Create: `tests/e2e/[feature-name].spec.ts`

- [ ] **Step 1: Write E2E test scenarios** — Cover each user story / acceptance criteria from the spec as a test scenario.
- [ ] **Step 2: Run E2E tests against running app** — Run: `[e2e test command]`. Expected: all pass.
- [ ] **Step 3: Commit** — `git commit -m "test: add E2E tests for [feature]"`

**Acceptance Criteria:**
- [ ] Every user-facing behavior from the spec has a corresponding E2E test
- [ ] Happy path passes
- [ ] Key error/failure scenarios are covered
- [ ] All E2E tests pass
```

## QA Case Updates

The plan **must** include a task that reviews and updates QA cases in `docs/qa/user-story/` to reflect the changed system behavior. If no existing QA case covers the new behaviors, add new ones.

```markdown
### Task N+3: QA — Update User Story Cases for [Feature Name]

**Definition:** Review existing QA test cases in `docs/qa/user-story/` for relevance to this feature. Update cases affected by changed system behavior, and add new cases for behaviors not yet covered. Then run `/qa` to verify the feature works end-to-end.

**Files:**
- Review: `docs/qa/user-story/` (all relevant case files)
- Modify: `docs/qa/user-story/<affected-case-file>.md` (update changed behaviors)
- Create: `docs/qa/user-story/<new-case-file>.md` (if new cases needed)

- [ ] **Step 1: Scan existing QA cases** — Read all case files in `docs/qa/user-story/`. Identify cases that reference changed components, APIs, or user flows.
- [ ] **Step 2: Assess coverage gaps** — For each acceptance criterion in the feature spec, determine if an existing QA case covers it. List gaps.
- [ ] **Step 3: Update affected cases** — For cases impacted by changed system behavior, update test steps and expected outcomes to match new behavior.
- [ ] **Step 4: Add new cases** — For acceptance criteria with no existing QA coverage, write new test cases following the established format in `docs/qa/user-story/`.
- [ ] **Step 5: Commit QA updates** — `git commit -m "qa: update user story cases for [feature]"`
- [ ] **Step 6: Run /qa** — Invoke the `/qa` skill against the running application to verify the feature works correctly and catch any issues missed by automated tests.
- [ ] **Step 7: Fix QA findings** — Address any bugs found during QA. Commit fixes.
- [ ] **Step 8: Commit final** — `git commit -m "fix: address QA findings for [feature]"` (if fixes were needed)

**Acceptance Criteria:**
- [ ] Every QA case affected by this feature has been updated to reflect new behavior
- [ ] Every acceptance criterion from the feature spec has QA case coverage (existing or new)
- [ ] `/qa` runs clean with no unresolved bugs related to this feature
```

## Self-Review

After writing the complete plan, review against the spec with fresh eyes:

**1. Spec coverage:** Can you point to a task for every spec requirement? List any gaps.

**2. Placeholder scan:** Search for red flags from the "No Placeholders" section. Fix them.

**3. E2E coverage:** Does the final E2E task cover every user-facing behavior from the spec?

**4. QA coverage:** Does the QA task cover every acceptance criterion? Are existing QA cases in `docs/qa/user-story/` reviewed for impact?

**5. Acceptance criteria check:** Does every task have clear, measurable acceptance criteria? Can an engineer verify completion unambiguously?

**6. Type consistency:** Do types, method signatures, and property names used in later tasks match earlier task definitions?

If you find issues, fix them inline. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
