---
name: new-oc-skill
description: Create new OpenClaw/IF skill in src/agent/skills/
---

# new-oc-skill

Create a new OpenClaw/IF skill in `src/agent/skills/` following the established patterns, and register it in OpenClaw configuration for agent use.

**User's instruction:** {{args}}

## Overview

This skill guides through creating a new Information Factory (IF) skill for OpenClaw Gateway. IF skills are Python-based CLI tools that integrate with the Information Factory backend API.

Reference guide: `@docs/knowledge/agentic/operations/how-to-add-an-if-skill.md`

## Your Task

1. Parse the user's request to extract skill details
2. Create the skill directory structure
3. Generate required files (SKILL.md, __init__.py, scripts)
4. Update configuration files
5. Provide setup and testing instructions

## Approach

### Step 1: Extract Skill Requirements

From the user's request, identify:
- **Skill name**: Directory name (kebab-case, e.g., "if-myskill")
- **Description**: What does this skill do?
- **Actions**: List of action names and descriptions
- **Target agent**: Which agent(s) will use this skill? (main, accountant, trader, etc.)
- **API endpoints**: Which IF Backend endpoints does it need?
- **Dependencies**: Special requirements (external APIs, libraries, etc.)

Ask the user for any missing critical information.

### Step 2: Create Skill Directory Structure

Create the directory at `src/agent/skills/{skill-name}/`:

```
src/agent/skills/{skill-name}/
├── SKILL.md                    # Skill metadata (required)
├── __init__.py                 # Package initialization (required)
├── TESTING.md                  # Testing documentation
├── api_client.py               # IF Backend API client (if needed)
├── config.py                   # Configuration and constants
├── security.py                 # Security utilities (if needed)
├── scripts/                    # Executable scripts
│   ├── action_name.py         # Main script files
│   └── ...
└── references/                 # Documentation
    ├── schema.md               # Database schema reference
    └── ...
```

### Step 3: Generate SKILL.md

Create `src/agent/skills/{skill-name}/SKILL.md`:

```markdown
# {Skill Name}

## Overview

{Detailed description of the skill's purpose}

## Actions

### action_name

{Description of what this action does}

**Usage:**
```bash
python scripts/action_name.py --param value
```

**Parameters:**
- `--param` - Description

**Returns:**
```json
{
  "success": true,
  "data": {...}
}
```

## Dependencies

- python3 (>= 3.11)
- OPENCLAW_AGENT_TOKEN (for API authentication)
- IF_BACKEND_URL (IF Backend API endpoint)

## Related Skills

- {Related skills if any}
```

### Step 4: Generate __init__.py

Create `src/agent/skills/{skill-name}/__init__.py`:

```python
"""{Skill Name} Skill for Information Factory."""

__version__ = "0.1.0"
```

### Step 5: Generate Script Template

For each action, create `src/agent/skills/{skill-name}/scripts/{action_name}.py`:

```python
#!/usr/bin/env python3
"""
Information Factory {Skill Name} - {Action Name}

{Description of what this action does}
"""

import argparse
import json
import sys
import os
from pathlib import Path

# Add parent directory to path for imports
sys.path.insert(0, str(Path(__file__).parent.parent))

try:
    from api_client import IFAPIClient
except ImportError:
    # Fallback if api_client doesn't exist
    IFAPIClient = None

def main():
    parser = argparse.ArgumentParser(
        description="{Action description}"
    )
    parser.add_argument(
        "--param",
        required=True,
        help="Parameter description"
    )
    args = parser.parse_args()

    try:
        # Your logic here
        if IFAPIClient:
            client = IFAPIClient()
            # Make API calls
            result = {"success": True, "data": {...}}
        else:
            result = {"success": True, "data": {...}}

        print(json.dumps(result, indent=2, default=str))
        return 0
    except Exception as e:
        error = {
            "success": False,
            "error": str(e)
        }
        print(json.dumps(error, indent=2))
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

**Make scripts executable:**
```bash
chmod +x src/agent/skills/{skill-name}/scripts/*.py
```

### Step 6: Update pyproject.toml (if skill has external dependencies)

If the skill requires external Python libraries, add them to `pyproject.toml`:

```toml
[project]
dependencies = [
    # ... existing dependencies ...

    # {Skill Name} dependencies
    "library-name>=version",  # Description of what library does
]
```

### Step 7: Update Docker Compose

Add the skill mount to `docker-compose.yml`:

```yaml
services:
  openclaw_gateway:
    volumes:
      - ./src/agent/skills/{skill-name}:/app/skills/{skill-name}:ro
```

**Note:** If the entire `./src/agent/skills` directory is already mounted, this step is not needed.

### Step 8: Update OpenClaw Configuration

Add the skill to `workspace/openclaw.json`:

```json
{
  "agents": {
    "list": [
      {
        "id": "{agent_id}",
        "skills": [
          "{skill-name}"
        ]
      }
    ]
  }
}
```

### Step 9: Rebuild Docker Image (if dependencies were added)

If you added new dependencies to `pyproject.toml`, rebuild the Docker image:

```bash
# Build the openclaw-python image
docker build -f Dockerfile.openclaw-python -t openclaw-python:local .

# Or use docker compose (if configured)
docker compose build openclaw-gateway
```

### Step 10: Restart Container

Restart the OpenClaw gateway to pick up changes:

```bash
docker compose up -d openclaw-gateway
```

### Step 11: Create Tests

Create test files:

```bash
# Unit test
touch test/unit_test/agent/skills/test_{skill_name}.py

# Integration test
touch test/integration_test/skills/test_{skill_name}_integration.py
```

### Step 12: Reload and Verify

```bash
# Reload skills (no downtime)
docker exec -it information_factory_openclaw_gateway openclaw reload

# Verify skill is loaded
docker exec -it information_factory_openclaw_gateway openclaw skills list | grep {skill-name}

# Check logs
docker logs information_factory_openclaw_gateway --tail 50

# Verify skill files exist in container
docker exec information_factory_openclaw_gateway ls -la /home/node/.openclaw/huotui/skills/{skill-name}/
```

### Step 13: Test the Skill

```bash
# Manual test from container
docker exec -it information_factory_openclaw_gateway python /app/skills/{skill-name}/scripts/{action_name}.py --param test

# Via API
curl -X POST http://localhost:18788/api/tools/execute \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "{skill-name}/{action_name}",
    "arguments": {"param": "value"}
  }'
```

## Input Format

```
/new-oc-skill create {skill-name} with actions {action1}, {action2} for {agent}
/new-oc-skill create skill called {name} that {description}
/new-oc-skill add {skill-name} to agent {agent-id}
```

**Examples:**
```
/new-oc-skill create if-notifier with actions send_notification, check_status for main
/new-oc-skill create skill called "if-analyzer" that analyzes data for accountant
/new-oc-skill add if-calculator to trader agent
```

## Output Format

Provide a comprehensive summary with:

### 1. Skill Structure

```
Created: src/agent/skills/{skill-name}/

Structure:
├── SKILL.md
├── __init__.py
├── api_client.py
├── config.py
├── scripts/
│   ├── {action1}.py
│   └── {action2}.py
└── references/
    └── schema.md
```

### 2. Configuration Changes

**Added to docker-compose.yml:**
```yaml
volumes:
  - ./src/agent/skills/{skill-name}:/app/skills/{skill-name}:ro
```

**Added to workspace/openclaw.json:**
```json
{
  "agents": {
    "list": [{
      "skills": ["{skill-name}"]
    }]
  }
}
```

### 3. Next Steps

1. ✅ Skill directory created
2. ✅ SKILL.md generated
3. ✅ Scripts templated
4. ⏳ Implement script logic
5. ⏳ Write tests
6. ⏳ Update pyproject.toml (if external dependencies)
7. ⏳ Update docker-compose.yml (if not already mounted)
8. ⏳ Update workspace/openclaw.json
9. ⏳ Rebuild Docker image (if dependencies added)
10. ⏳ Restart container
11. ⏳ Reload: `docker exec -it information_factory_openclaw_gateway openclaw reload`
12. ⏳ Verify skill is loaded
13. ⏳ Test manually

### 4. Implementation Checklist

- [ ] Implement action scripts with actual logic
- [ ] Add error handling and validation
- [ ] Write unit tests
- [ ] Write integration tests
- [ ] Run tests: `python scripts/test/run_tests.py test test/unit_test/agent/skills/test_{skill_name}.py`
- [ ] Add dependencies to pyproject.toml (if external libraries needed)
- [ ] Update docker-compose.yml mount (if not already mounted)
- [ ] Update workspace/openclaw.json
- [ ] Rebuild Docker image (if dependencies added): `docker build -f Dockerfile.openclaw-python -t openclaw-python:local .`
- [ ] Restart container: `docker compose up -d openclaw-gateway`
- [ ] Reload OpenClaw skills: `docker exec -it information_factory_openclaw_gateway openclaw reload`
- [ ] Verify skill appears in list: `docker exec information_factory_openclaw_gateway openclaw skills list | grep {skill-name}`
- [ ] Manual testing completed
- [ ] Documentation created

## Examples

### Example 1: Full Skill Creation

**User input:**
```
/new-oc-skill create if-messenger with actions send_message, get_history for main
```

**Output:**
```
Creating IF skill "if-messenger" with 2 actions for agent "main"...

1. Created directory structure:
   src/agent/skills/if-messenger/
   ├── SKILL.md
   ├── __init__.py
   ├── api_client.py
   ├── scripts/
   │   ├── send_message.py
   │   └── get_history.py
   └── references/
       └── schema.md

2. Configuration changes needed:

   docker-compose.yml:
   ```yaml
   volumes:
     - ./src/agent/skills/if-messenger:/app/skills/if-messenger:ro
   ```

   workspace/openclaw.json:
   ```json
   {
     "agents": {
       "list": [{
         "id": "main",
         "skills": [..., "if-messenger"]
       }]
     }
   }
   ```

3. Next steps:
   - Implement send_message.py logic
   - Implement get_history.py logic
   - Write tests
   - Update docker-compose.yml (if not already mounted)
   - Update workspace/openclaw.json
   - Rebuild Docker image (if dependencies added)
   - Restart container: docker compose up -d openclaw-gateway
   - Reload: docker exec -it information_factory_openclaw_gateway openclaw reload

4. To test:
   docker exec -it information_factory_openclaw_gateway python /app/skills/if-messenger/scripts/send_message.py --recipient 123 --body "test"
```

### Example 2: Minimal Input

**User input:**
```
/new-oc-skill create if-helper
```

**Output:**
```
I need more details to create the if-helper skill.

Please provide:
1. **Actions**: What actions should this skill have? (e.g., query, update, process)
2. **Target agent**: Which agent should use this skill? (main, accountant, trader)
3. **Purpose**: What does this skill do?

Example: /new-oc-skill create if-helper with actions check_status for accountant
```

## Reference Patterns

Use existing skills as templates:

| Skill | Pattern | Location |
|-------|---------|----------|
| **if-database** | Database CRUD operations | `@src/agent/skills/if-database/` |
| **if-memory** | Vector search and memory | `@src/agent/skills/if-memory/` |
| **if-accountant** | Document processing | `@src/agent/skills/if-accountant/` |
| **if-ibclient** | External API integration | `@src/agent/skills/if-ibclient/` |

## Best Practices

1. **API Client**: Use `api_client.py` for IF Backend communication
2. **Error Handling**: Always handle exceptions and return JSON
3. **Input Validation**: Validate arguments before processing
4. **Security**: Never log sensitive data (tokens, passwords)
5. **Testing**: Write both unit and integration tests
6. **Documentation**: Keep SKILL.md up to date
7. **Executable**: Make all scripts executable with `chmod +x`

## Common Script Patterns

### Handling External Dependencies

If your skill uses external Python libraries:

1. **Add to pyproject.toml:**
```toml
[project]
dependencies = [
    # ... existing dependencies ...

    # {Skill Name} dependencies
    "library-name>=version",  # Description
]
```

2. **Rebuild Docker image:**
```bash
docker build -f Dockerfile.openclaw-python -t openclaw-python:local .
```

3. **Restart container:**
```bash
docker compose up -d openclaw-gateway
```

4. **Verify installation:**
```bash
docker exec information_factory_openclaw_gateway /home/node/.venv/bin/python -c "import library_name; print(library_name.__version__)"
```

**Important - Script Naming:**
- Never name your script the same as an imported library (e.g., don't name a script `yfinance.py` if you import `yfinance`)
- Use a prefix like `if-{library}.py` to avoid module shadowing

### Simple API Call

### Simple API Call
```python
client = IFAPIClient()
result = client.get(f"/api/resource/{args.id}")
print(json.dumps({"success": True, "data": result}))
```

### Data Processing
```python
data = process_input(args.input)
result = {
    "success": True,
    "processed": data
}
print(json.dumps(result))
```

### File Processing
```python
file_path = Path(args.file)
content = file_path.read_text()
result = process_content(content)
print(json.dumps({"success": True, "data": result}))
```

## Troubleshooting

**Skill not found after reload:**
- Check SKILL.md exists in skill root
- Verify docker-compose.yml mount
- Check container permissions: `docker exec information_factory_openclaw_gateway ls -la /app/skills/{skill-name}/`

**Scripts won't execute:**
- Make executable: `chmod +x scripts/*.py`
- Check shebang: `#!/usr/bin/env python3`

**Import errors or "module has no attribute":**
- Check script name doesn't shadow imported library (e.g., `yfinance.py` when importing `yfinance`)
- Rename script to avoid conflicts: use prefix like `if-{library}.py`

**API connection failed:**
- Verify IF_BACKEND_URL environment variable
- Check backend is running
- Verify OPENCLAW_AGENT_TOKEN is set

**Container doesn't have new dependencies:**
- Rebuild Docker image after updating pyproject.toml
- Restart container after rebuild

## Related Documentation

- Complete guide: `@docs/knowledge/agentic/operations/how-to-add-an-if-skill.md`
- OpenClaw docs: `@deps/openclaw/docs/`
- Existing skills: `@src/agent/skills/`
