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

Or use the gateway tool if restart permission is enabled in config.

## Updating Existing Provider

### Update API Key

Edit `auth-profiles.json` and change the `key` field for the target provider profile.

### Update Model Configuration

Edit `openclaw.json` in the `models.providers` section and modify the relevant fields.

### Update Alias

Edit `openclaw.json` in the `agents.defaults.models` section.

After any changes, restart OpenClaw to apply.

## Removing a Provider

### Remove Authentication

Delete the provider entry from `auth-profiles.json`:

```json
{
  "profiles": {
    "provider-to-remove:default": { ... }  // Delete this
  }
}
```

### Remove Model Definition

Delete the provider entry from `openclaw.json` `models.providers`:

```json
{
  "models": {
    "providers": {
      "provider-to-remove": { ... }  // Delete this
    }
  }
}
```

### Remove Aliases

Delete any aliases in `agents.defaults.models` that reference the removed provider.

After removal, restart OpenClaw.

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
- Claude proxies

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
