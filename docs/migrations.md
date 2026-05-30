# Migration Guides

> **Last updated:** 2026-05-30

## Migrating to Claude 4.x Models

### From claude-sonnet-4-5 → claude-sonnet-4-6

```python
# Before
model="claude-sonnet-4-5-20250929"

# After
model="claude-sonnet-4-6"
```

No API changes required. Drop-in replacement with improved capabilities.

### New in claude-opus-4-8 (v0.105.0, 2026-05-28)

- Mid-conversation system blocks
- Usage token details
- Custom file size caps

### From Claude 3.x → Claude 4.x

Claude 4.x models are API-compatible. Just update the model ID:

```python
# Before
model="claude-3-5-sonnet-20241022"

# After
model="claude-sonnet-4-6"
```

**Verify behavior changes:**
- Response formatting may differ
- Extended thinking now available on Sonnet (was Opus-only in 3.x)
- Tool use behavior improved in 4.x

## Migrating from Text Completions API

The legacy `/v1/complete` endpoint is deprecated. Migrate to `/v1/messages`:

```python
# DEPRECATED
response = client.completions.create(
    model="claude-instant-1",
    prompt="\n\nHuman: Hello\n\nAssistant:",
    max_tokens_to_sample=1024,
)
text = response.completion

# NEW
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
text = response.content[0].text
```

**Key differences:**
- No `\n\nHuman:` / `\n\nAssistant:` formatting needed
- `messages` array replaces `prompt`
- `max_tokens` replaces `max_tokens_to_sample`
- Response in `content[0].text` not `.completion`

## Model ID Reference

| Old ID | New ID | Notes |
|--------|--------|-------|
| `claude-3-5-sonnet-20241022` | `claude-sonnet-4-6` | Use 4.x for latest |
| `claude-3-5-haiku-20241022` | `claude-haiku-4-5-20251001` | Use 4.x for latest |
| `claude-3-opus-20240229` | `claude-opus-4-8` | Use 4.x for latest |
| `claude-instant-1` | `claude-haiku-4-5-20251001` | Deprecated |
| `claude-2.1` | `claude-sonnet-4-6` | Deprecated |

## SDK Version Migrations (Python)

| Version | Change |
|---------|--------|
| v0.100.0+ | Managed Agents multiagents, webhooks, vault validation |
| v0.101.0+ | AWS client for Claude Platform on AWS |
| v0.102.0+ | Cache diagnostics beta support |
| v0.103.0+ | Self-hosted sandboxes in Managed Agents |
| v0.104.0+ | `thinking-token-count` beta for streaming |
| v0.105.0+ | claude-opus-4-8 support, mid-conversation system blocks |

## Tool Use Migration

Current format for tool definitions (use `input_schema`, not `parameters`):

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get weather for a location",
        "input_schema": {          # Note: input_schema
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"]
        }
    }
]

# Tool results format
{
    "role": "user",
    "content": [
        {
            "type": "tool_result",    # Note: tool_result type
            "tool_use_id": tool.id,
            "content": [{"type": "text", "text": result}]
        }
    ]
}
```

## Related

- [Models](./MODELS.md)
- [Authentication](./authentication.md)
- [SDKs](./sdks.md)
- [Prompt Caching](./prompt-caching.md)
