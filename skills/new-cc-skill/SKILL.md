---
name: new-cc-skill
description: Claude Code skill generator - create new skills
---

# new-cc-skill

You are a Claude Code skill generator. Create a new Claude Code skill based on the user's requirements.

**User's request:** {{args}}

## Your Task

Create a new Claude Code skill in `.claude/skills/{skill-name}/SKILL.md` format based on the user's description.

## Steps

### 1. Parse the Request

Extract from the user's request:
- **Skill name**: The command name (e.g., "new-cc-skill", "openclaw")
- **Description**: What does this skill do?
- **Purpose**: When should this skill be used?
- **Input**: What arguments or parameters does it accept?

### 2. Generate the Skill File

Create a directory at `.claude/skills/{skill-name}/` and create `SKILL.md` inside with the following structure:

```markdown
---
name: {skill-name}
description: {Brief description of what this skill does}
---

# {skill-name}

{Brief description of what this skill does}

**User's instruction:** {{args}}

## Your Task

{What the skill should accomplish when invoked}

## Approach

1. **Step 1** - {Description}
2. **Step 2** - {Description}
3. **Step 3** - {Description}

## Input Format

{Describe expected arguments or input format}

## Output Format

{Describe expected output format}

## Examples

{Provide concrete examples of usage}
```

### 3. Best Practices

When creating the skill:

- **Keep it focused**: Each skill should have a single, clear purpose
- **Be specific**: Use clear, actionable instructions
- **Include examples**: Show concrete usage patterns
- **Reference context**: Use `@` references to point to relevant files (e.g., `@docs/`, `@src/`)
- **Use templates**: Include `{{args}}` placeholder for user arguments
- **Think chain-of-thought**: Structure the skill with clear steps for the AI to follow

### 4. Skill Categories

Consider common skill patterns:

**Documentation skills**: Reference specific parts of the codebase
- Example: `openclaw` - Answer questions about OpenClaw

**Workflow skills**: Guide through multi-step processes
- Example: `plan` - Create implementation plans
- Example: `feature` - Implement features from design

**Analysis skills**: Analyze code or configurations
- Example: `review` - Review code changes
- Example: `debug` - Debug issues

**Generation skills**: Generate code or artifacts
- Example: `spec` - Generate specifications
- Example: `test` - Generate tests

### 5. Validate Before Creating

Before writing the skill file, confirm:
- [ ] Skill name is clear and memorable
- [ ] Purpose is well-defined
- [ ] Input/output format is specified
- [ ] Examples are included
- [ ] File reference pattern (`@`) is used if needed

## Output

After creating the skill, provide a summary:

```
Created: .claude/skills/{skill-name}/SKILL.md

Name: {skill-name}
Description: {brief description}
Usage: /{skill-name} {example-args}
```

## Example

If the user requests: "Create a skill called 'git-status' that shows git status with analysis"

You would create `.claude/skills/git-status/SKILL.md` with instructions to:
1. Run `git status`
2. Analyze the output
3. Provide insights about changed files, branches, etc.
4. Suggest next actions
