---
name: model-provider-manager
description: Add, update, or remove AI model providers and models in OpenClaw configuration. Use when the user wants to configure new API providers (like OpenRouter, Anthropic, OpenAI, etc.), add new models, update API keys, or modify existing provider configurations.
---

# Model Provider Manager

This skill provides step-by-step guidance for managing AI model providers and models in OpenClaw.

## Overview

OpenClaw separates authentication from model configuration:
- **Authentication**: Stored in `~/.openclaw/agents/main/agent/auth-profiles.json`
- **Model definitions**: Stored in `~/.openclaw/openclaw.json`

**Critical**: Both files must be kept in sync. When removing a provider, you must clean up BOTH files.

## Adding a New Provider and Model

### Step 1: Add Authentication Profile

Edit `~/.openclaw/agents/main/agent/auth-profiles.json` and add the new provider profile:

```json
{
  "version": 1,
  "profiles": {
    "existing-provider:default": { ... },
    "new-provider:default": {
      "type": "api_key",
      "provider": "new-provider",
      "key": "your-api-key-here"
    }
  }
}
```

**Authentication types:**
- `api_key`: For API key authentication (most common)
- `oauth`: For OAuth authentication (includes `access`, `refresh`, `expires` fields)

**Note:** Do NOT manually add `lastGood` or `usageStats` - these are auto-generated at runtime.

### Step 2: Add Model Definition

Edit `~/.openclaw/openclaw.json` in the `models.providers` section:

```json
{
  "models": {
    "providers": {
      "new-provider": {
        "baseUrl": "https://api.provider.com/v1",
        "api": "openai-completions",
        "models": [
          {
            "id": "model-id",
            "name": "Friendly Model Name",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 128000,
            "maxTokens": 4096
          }
        ]
      }
    }
  }
}
```

**API types:**
- `openai-completions`: OpenAI-compatible API (most common)
- `anthropic-messages`: Anthropic Claude API format

**Input types:**
- `["text"]`: Text-only models
- `["text", "image"]`: Multimodal models

### Step 3: Add Model Alias (Optional)

In `openclaw.json`, add an alias in `agents.defaults.models`:

```json
{
  "agents": {
    "defaults": {
      "models": {
        "new-provider/model-id": {
          "alias": "short-name"
        }
      }
    }
  }
}
```

### Step 4: Restart OpenClaw

After configuration changes, restart OpenClaw:

```bash
openclaw gateway restart
```

Or use the `exec` tool to run the restart command.

## Updating Existing Provider

### Update API Key

Edit `auth-profiles.json` and change the `key` field for the target provider profile, then restart.

### Update Model Configuration

Edit `openclaw.json` in the `models.providers` section and modify the relevant fields, then restart.

### Update Alias

Edit `openclaw.json` in the `agents.defaults.models` section, then restart.

### Change Default Model

Edit `openclaw.json` and update `agents.defaults.model.primary`:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "provider-name/model-id"
      }
    }
  }
}
```

Then restart OpenClaw.

## Removing a Provider

**Important**: You must clean up BOTH configuration files when removing a provider.

### Step 1: Remove from openclaw.json

1. Remove the provider from `models.providers` section
2. Remove any model aliases in `agents.defaults.models` that reference this provider
3. If this provider's model is the default, change `agents.defaults.model.primary` to another model

### Step 2: Remove from auth-profiles.json

1. Remove the provider profile from `profiles` section
2. Remove the provider from `lastGood` section (if present)
3. Remove the provider's usage stats from `usageStats` section (if present)

### Step 3: Restart OpenClaw

After cleaning up both files, restart OpenClaw to apply changes.

### Example: Complete Removal

**Before removal, check both files:**

```bash
# Read current config
read path:~/.openclaw/openclaw.json
read path:~/.openclaw/agents/main/agent/auth-profiles.json
```

**Remove from openclaw.json:**

```json
{
  "models": {
    "providers": {
      "provider-to-remove": { ... }  // Delete this entire block
    }
  },
  "agents": {
    "defaults": {
      "models": {
        "provider-to-remove/model-id": { ... }  // Delete this
      }
    }
  }
}
```

**Remove from auth-profiles.json:**

```json
{
  "profiles": {
    "provider-to-remove:default": { ... }  // Delete this
  },
  "lastGood": {
    "provider-to-remove": "..."  // Delete this
  },
  "usageStats": {
    "provider-to-remove:default": { ... }  // Delete this
  }
}
```

**Then restart:**

```bash
openclaw gateway restart
```

## File Locations

- **Windows**: 
  - Auth: `C:\Users\<username>\.openclaw\agents\main\agent\auth-profiles.json`
  - Config: `C:\Users\<username>\.openclaw\openclaw.json`
  
- **Linux/Mac**:
  - Auth: `~/.openclaw/agents/main/agent/auth-profiles.json`
  - Config: `~/.openclaw/openclaw.json`

## Common Providers

### OpenAI-compatible APIs
Most providers use `"api": "openai-completions"`:
- OpenRouter
- Together AI
- Groq
- DeepSeek
- Custom OpenAI proxies

### Anthropic-compatible APIs
Use `"api": "anthropic-messages"`:
- Anthropic Claude (direct)
- Claude proxies (e.g., PackyAPI, Aiberm)

## Workflow Best Practices

1. **Always read both files first** before making changes
2. **Make changes to both files** when adding/removing providers
3. **Validate JSON syntax** before saving (no trailing commas!)
4. **Restart OpenClaw** after every configuration change
5. **Check logs** after restart to ensure no errors

## Troubleshooting

**Issue**: Restart command fails with "disabled" error

**Solution**: Enable restart permission in `openclaw.json`:
```json
{
  "commands": {
    "restart": true
  }
}
```

**Issue**: Model not appearing after restart

**Solution**: Check that:
1. Provider name matches between auth-profiles.json and openclaw.json
2. JSON syntax is valid (no trailing commas, proper quotes)
3. OpenClaw restarted successfully (check logs)

**Issue**: Authentication errors when using model

**Solution**: Verify:
1. API key is correct in auth-profiles.json
2. Provider's baseUrl is correct
3. API type matches provider's actual API format

**Issue**: Old provider still showing up after removal

**Solution**: You likely only removed it from one file. Check BOTH:
1. `openclaw.json` - remove from `models.providers` and `agents.defaults.models`
2. `auth-profiles.json` - remove from `profiles`, `lastGood`, and `usageStats`
3. Restart OpenClaw again

## Agent Implementation Notes

When implementing this skill:

1. **Use `read` tool** to load both config files first
2. **Use `write` tool** to update files (overwrites entire file)
3. **Use `exec` tool** to restart: `openclaw gateway restart`
4. **Wait for restart** to complete before confirming success
5. **Provide clear feedback** about what was changed

Example implementation flow:
```
1. read openclaw.json
2. read auth-profiles.json
3. write openclaw.json (with changes)
4. write auth-profiles.json (with changes)
5. exec: openclaw gateway restart
6. confirm changes to user
```
