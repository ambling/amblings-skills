---
name: new-tg-bot
description: Create Telegram bot and bind to OpenClaw agent
---

# new-tg-bot

Create a new Telegram bot and bind it to an OpenClaw agent, following the multi-account pattern established for the accountant bot.

**User's request:** {{args}}

## Overview

This skill guides through creating a new Telegram bot via BotFather, adding the bot token to the environment, configuring OpenClaw multi-account Telegram settings, and binding the bot to a specific agent.

Reference the existing accountant setup in `@workspace/openclaw.json` for the pattern.

## Your Task

1. Parse the user's request to extract:
   - Bot account ID (e.g., "trader", "support")
   - Target agent ID (must exist in `agents.list`)
   - Bot token (if provided, or guide to create one)
   - Allowed user IDs (optional)

2. Update the configuration files

3. Provide setup instructions

## Approach

### Step 1: Extract Requirements

From the user's request, identify:
- **Account ID**: The identifier for this bot account (lowercase, hyphens allowed)
- **Agent ID**: The OpenClaw agent to bind this bot to (must exist in `agents.list`)
- **Bot Token**: The Telegram bot token from @BotFather
- **Allowed Users**: Telegram user IDs allowed to DM this bot (default: existing allowlist)

Ask the user for any missing information.

### Step 2: Validate Agent Exists

Check `@workspace/openclaw.json` under `agents.list` to confirm the target agent exists.

Available agents in current config:
- `main` (Huotui) - Default agent
- `accountant` (Accountant) - Financial document processing
- `trader` (Trader) - Trading operations

If the agent doesn't exist, inform the user they need to create it first or choose an existing agent.

### Step 3: Add Bot Token to .env

Add the bot token to `@.env`:
```bash
TELEGRAM_{ACCOUNT_ID}_BOT_TOKEN={bot_token}
```

Example for trader bot:
```bash
TELEGRAM_TRADER_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
```

### Step 4: Update openclaw.json

Update `@workspace/openclaw.json`:

**Add to `channels.telegram.accounts`:**
```json
{
  "accounts": {
    "{account_id}": {
      "botToken": "${TELEGRAM_{ACCOUNT_ID}_BOT_TOKEN}",
      "dmPolicy": "allowlist",
      "allowFrom": [6475393362],
      "groupPolicy": "allowlist",
      "streamMode": "partial"
    }
  }
}
```

**Add to `bindings`:**
```json
{
  "bindings": [
    {
      "match": {
        "channel": "telegram",
        "accountId": "{account_id}"
      },
      "agentId": "{agent_id}"
    }
  ]
}
```

### Step 5: Provide Summary

Show the user:
1. What was added to `.env`
2. What was added to `openclaw.json`
3. Next steps (restart OpenClaw, test the bot)

## Input Format

```
/new-tg-bot <account-id> for <agent-id> with token <bot-token>
/new-tg-bot bind <account-id> to <agent-id>
/new-tg-bot create bot for <agent-id>
```

**Examples:**
```
/new-tg-bot trader-bot for trader with token 123456:ABC
/new-tg-bot support for main
/new-tg-bot create a new bot called "helper" for the accountant agent
```

## Output Format

Provide a clear summary with:

### 1. Configuration Changes

**Added to `.env`:**
```bash
TELEGRAM_{ACCOUNT}_BOT_TOKEN=...
```

**Added to `workspace/openclaw.json`:**
- Telegram account under `channels.telegram.accounts.{account_id}`
- Binding under `bindings[]`

### 2. Bot Information

- **Account ID**: {account_id}
- **Agent**: {agent_id}
- **Bot Token**: {token} (or env var name)

### 3. Next Steps

1. Verify `.env` has the token
2. Restart OpenClaw gateway
3. Start a chat with the bot on Telegram
4. Verify it responds as the correct agent

## Examples

### Example 1: Full Setup

**User input:**
```
/new-tg-bot create a bot called "trader" for the trader agent with token 123456789:ABCdefGHI
```

**Output:**
```
Creating Telegram bot "trader" for agent "trader"...

1. Added to .env:
TELEGRAM_TRADER_BOT_TOKEN=123456789:ABCdefGHI

2. Added to workspace/openclaw.json:
{
  "channels": {
    "telegram": {
      "accounts": {
        "trader": {
          "botToken": "${TELEGRAM_TRADER_BOT_TOKEN}",
          "dmPolicy": "allowlist",
          "allowFrom": [6475393362],
          "groupPolicy": "allowlist",
          "streamMode": "partial"
        }
      }
    }
  },
  "bindings": [
    {
      "match": { "channel": "telegram", "accountId": "trader" },
      "agentId": "trader"
    }
  ]
}

Next steps:
1. Restart OpenClaw: docker compose restart openclaw
2. Find your bot on Telegram and start a chat
3. Send a message to verify it responds as the Trader agent
```

### Example 2: Minimal Input

**User input:**
```
/new-tg-bot helper for accountant
```

**Output:**
```
I need the bot token to set up the helper bot.

Please:
1. Chat with @BotFather on Telegram
2. Run /newbot
3. Save the token
4. Run: /new-tg-bot helper for accountant with token <your-token>
```

## Reference Pattern

This skill follows the pattern established for the accountant bot:

**Existing setup in `workspace/openclaw.json`:**
- `channels.telegram.accounts.accountant` → Agent: `accountant`
- `channels.telegram.accounts.huotui` → Agent: `main`
- `bindings[]` maps account IDs to agent IDs

**Key points:**
- Use `accounts` object for multi-bot setup
- Use environment variables for tokens (not hardcoded)
- Use `bindings` to route accounts to agents
- Each bot has independent session storage
