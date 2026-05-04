---
name: report
description: Reporting phase - document implementation status
---

# report

You are now in the reporting phase. Your task is to create or update a file under docs/implementation/<feature_name>/<component>/ to report the implementation status of the current feature or component. Your specific instruction is: '{{args}}'. The report should include any blocked items and next steps. Following @docs/knowledge/agentic/coding/engineering.md for how to organize the docs. When you have done the work, request user to review and give you feedback.

## Progress Tracking

### Update Notes File

Create/update `docs/coordinate/<feature>/report_notes.md`:

```markdown
# Report Notes: <Feature Name>

**Status**: COMPLETED
**Completed**: [timestamp]
**Report File**: [path to report file]

### Summary
[Brief summary of feature status]

### Overall Status
- **Phases Complete**: [N]/[M]
- **Tests**: [All passing / X failing]
- **Blockers**: [None / List of blockers]

### Next Steps
[Recommended next actions]

### Issues
[Any blockers]
```

### Output Status Report

After completion, provide:

```markdown
## Report Phase Complete

**Feature**: <feature>
**Status**: COMPLETED
**Report**: [path to report file]

### Summary
- [Brief summary of feature status]

### Overall Status
- **Phases Complete**: [N]/[M]
- **Tests**: [All passing / X failing]
- **Blockers**: [None / List of blockers]

### Next Steps
- [Recommended next actions]
```
