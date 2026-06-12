---
name: writing-specs
description: Use after brainstorming and discussion to formalize decisions into a feature spec and design spec before writing implementation plans
---

# Writing Specs

After brainstorming and sufficient discussion, summarize decisions into two structured specs that `writing-plans` will consume as input.

**Announce at start:** "I'm using the writing-specs skill to formalize the feature and design specs."

**Save specs to:** `docs/specs/YYYY-MM-DD-<feature-name>/spec.md` and `docs/specs/YYYY-MM-DD-<feature-name>/design.md`

## When to Use

- After brainstorming or design discussion has reached sufficient clarity
- Before invoking `writing-plans`
- When the user asks to formalize, document, or spec out what was discussed

## Prerequisites

Before writing specs, confirm:
- The core problem and goal are clear from discussion
- Key architectural decisions have been made (not still open)
- User has reviewed and agreed on the approach

If any of these are missing, surface the gap and ask before proceeding. Do not speculate to fill gaps.

## File Structure

Create a feature directory in `docs/specs/` with two files:

```
docs/specs/2026-06-06-notification-panel/
  spec.md
  design.md
```

Use today's date and a short kebab-case feature name as the directory name.

---

## Feature Spec Template

**File:** `spec.md`

**Purpose:** Define WHAT to build, for whom, and how to verify it works.

```markdown
# <Feature Name> — Feature Spec

**Date:** YYYY-MM-DD
**Status:** Draft | Approved
**Author:** <who requested this>

## Problem

[1-3 sentences: what problem exists today that this feature solves. Be specific about user pain, not abstract goals.]

## Goal

[1-2 sentences: what the world looks like after this feature ships. Concrete and measurable where possible.]

## User Stories

### US-1: <Story Title>
**As a** <role>, **I want to** <action> **so that** <benefit>.

**Happy path:** [Step-by-step what the user does and sees]
**Edge cases:** [What happens with bad input, missing data, concurrent actions]

### US-2: <Story Title>
[Repeat per distinct user interaction]

## Current Status

[What exists today — relevant features, partial implementations, workarounds users have. Omit if greenfield.]

## Acceptance Criteria

### Functional
- [ ] <Measurable condition — e.g., "User sees notification count badge update within 2 seconds of new event">
- [ ] <Measurable condition — e.g., "Clicking notification marks it as read and navigates to source">
- [ ] <One criterion per user story, at minimum>

### Non-Functional
- [ ] <Performance — e.g., "Notification list loads in <500ms with 1000 items">
- [ ] <Security — e.g., "User cannot access another user's notifications">
- [ ] <Compatibility — e.g., "Works on mobile viewport ≥375px">
- [ ] <Reliability — e.g., "Notification delivery is at-least-once; duplicates are deduped client-side">
```

---

## Design Spec Template

**File:** `design.md`

**Purpose:** Define HOW to build it — current system state, architecture changes, schemas, APIs.

```markdown
# <Feature Name> — Design Spec

**Date:** YYYY-MM-DD
**Status:** Draft | Approved
**Depends on:** <Feature spec file name>

## Current System

[Describe the relevant parts of the system as it exists today. Architecture, modules, data flow, database tables, APIs involved. Enough context for someone unfamiliar to understand what's changing.]

## Proposed Architecture

[2-4 sentences on the high-level approach. What new components are added, what existing ones change, and how they connect.]

## Modules

### <New/Changed Module Name>
**Responsibility:** [One sentence on what this module does]
**Location:** `path/to/module/`
**Dependencies:** [What other modules or services it depends on]

### <Existing Module Being Modified>
**Change:** [What specifically changes and why]
**Location:** `path/to/module/`
**Impact:** [What else depends on this module and might be affected]

## Schema Changes

### <New Table or Collection>
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | UUID | PK, auto-generated | Unique identifier |
| ... | ... | ... | ... |

**Indexes:** [List new indexes and why — e.g., "user_id + created_at for notification list query performance"]

### <Modified Table>
| Change | Field | Before | After | Reason |
|--------|-------|--------|-------|--------|
| Added | field_name | — | type | why |

## APIs

### <Endpoint or Function Name>
**Method:** GET | POST | PUT | DELETE
**Path:** `/api/v1/resource`
**Auth:** [Required role/permission]
**Request:**
- `field_name` (type, required) — description
**Response:**
- `field_name` (type) — description
**Error codes:**
- 400: [when]
- 403: [when]
- 404: [when]

## Protocols / Integrations

[WebSocket events, message queue topics, webhook payloads, third-party API calls. Omit section if none.]

## Migration Strategy

[How to roll this out without breaking existing functionality. Order of operations. Rollback plan if applicable.]

## Out of Scope

[Explicitly list what this design does NOT cover. Prevents scope creep during planning.]
```

---

## Writing Guidelines

**Do:**
- Be concise — every sentence should earn its place
- Use specific, measurable language in acceptance criteria
- Number and name user stories for traceability
- Map every functional acceptance criterion to a design change
- Include rollback/migration strategy in the design spec

**Do not:**
- Include code samples — specs describe intent, not implementation
- Use vague terms — "improve performance" → "reduce p95 latency from 2s to <500ms"
- Leave placeholders (TBD, TODO, "figure out later") — resolve open questions before writing specs
- Repeat information across the two specs — feature spec owns the WHAT, design spec owns the HOW
- Speculate — if a decision is open, flag it clearly and ask the user

## Self-Review

After writing both specs, verify:

**1. Feature completeness:** Does every discussed requirement have a corresponding user story or acceptance criterion?

**2. Design traceability:** Can you trace each functional acceptance criterion from the feature spec to a specific change in the design spec?

**3. No placeholders:** Search both files for TBD, TODO, "TBD", "figure out", "decide later". Fix all of them.

**4. No code samples:** Remove any code blocks. Specs describe intent, not implementation.

**5. Measurable criteria:** Are all non-functional requirements quantified? "Fast" is not a criterion. "<500ms p95" is.

**6. Scope boundary:** Does the design spec explicitly list what is out of scope?

If issues found, fix them before presenting to the user.

## User Review Gate

Present both specs to the user for review. The user must explicitly approve before proceeding.

If the user requests changes, update the specs and re-present. Do not proceed to `writing-plans` without approval.

## Handoff

After user approval:

**"Specs approved and saved to `docs/specs/`. Ready to create implementation plan? I'll invoke `writing-plans` which reads both specs to generate tasks."**

If approved, invoke `writing-plans` with the spec paths as context.

If the user wants to hold, commit the specs and tell them to invoke `writing-plans` when ready.
