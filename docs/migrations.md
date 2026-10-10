# Migration Guides

> **Last updated:** 2026-10-10

## Migrating to Claude 4.x Models

### From claude-haiku-4-5 → claude-haiku-5-5

> **Haiku 5.5 launched Oct 7, 2026.** Haiku 4.5 (`claude-haiku-4-5-20251001`) is now a legacy model, still Active on the deprecations page with retirement not sooner than Oct 15, 2026; no deprecation notice yet. Follow the official [Haiku 5.5 migration guide](https://platform.claude.com/docs/en/models/haiku-5-5/migration-guide). It is **not** a drop-in swap.

| Platform               | Claude Haiku 4.5                                  | Claude Haiku 5.5             |
| ---------------------- | ------------------------------------------------- | ---------------------------- |
| Claude API             | `claude-haiku-4-5-20251001` or `claude-haiku-4-5` | `claude-haiku-5-5`           |
| Amazon Bedrock         | `anthropic.claude-haiku-4-5`                      | `anthropic.claude-haiku-5-5` |
| Claude Platform on AWS | `claude-haiku-4-5`                                | `claude-haiku-5-5`           |
| Google Cloud           | `claude-haiku-4-5@20251001`                       | `claude-haiku-5-5`           |
| Microsoft Foundry      | `claude-haiku-4-5`                                | `claude-haiku-5-5`           |

Checklist from the migration guide (Haiku 4.5 starting point):

1. Swap the model ID (table above).
2. Recount prompts and revisit `max_tokens` and cost estimates: about 30% more tokens for the same text, more visual tokens for large images, and higher prices for prompts over 100,000 tokens.
3. Replace `thinking: {"type": "enabled", "budget_tokens": N}` with `{"type": "adaptive"}` and steer with `output_config.effort`.
4. Select content blocks by `type`; responses can start with `thinking` blocks.
5. Remove `temperature`, `top_p` and `top_k`.
6. Replace assistant prefill; end `messages` with a user turn.
7. Computer use: replace `computer_20250124` with `computer_toolset_20260801` (Claude API, Google Cloud) or `computer_20251124` + `computer-use-2025-11-24` (Amazon Bedrock).
8. Replay stored conversations through the account that produced them.
9. Keep conversations append-only if you send thinking blocks back.
10. Handle `stop_reason: "refusal"` (no server-side fallback on Haiku 5.5).
11. On Amazon Bedrock, structured outputs aren't available for Haiku 5.5: describe the format in the prompt or use a non-`strict` tool and validate.

Priority Tier is not supported on Haiku 5.5. Coming from Haiku 3.5 or Haiku 3 (both retired on the Claude API), also move to `code_execution_20250825`+ and `text_editor_20250728`, and handle the `refusal` and `model_context_window_exceeded` stop reasons. See [MODELS.md](./MODELS.md) ("Haiku 5.5 API Changes") for the full list.

```python
# Before
client.messages.create(
    model="claude-haiku-4-5",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    messages=[{"role": "user", "content": "..."}],
)

# After
client.messages.create(
    model="claude-haiku-5-5",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "medium"},
    messages=[{"role": "user", "content": "..."}],
)
```

### From claude-sonnet-4-5 → claude-sonnet-5-5 (Sonnet 4.5 deprecated)

> **Deprecated Sep 30, 2026:** `claude-sonnet-4-5-20250929` retires on the Claude API on **Nov 30, 2026**. The official recommended replacement is `claude-sonnet-5-5` (source: platform.claude.com/docs/en/about-claude/model-deprecations). This is **not** a drop-in swap; follow the official [Sonnet 5.5 migration guide, "Migrating from Claude Sonnet 4.5 or earlier"](https://platform.claude.com/docs/en/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45). Changes it lists include: prefill, `thinking: {"type": "enabled", "budget_tokens": N}` and non-default `temperature` / `top_p` / `top_k` return 400 errors; requests with no `thinking` field now run with adaptive thinking; Sonnet 4.5 has no effort parameter, so set one explicitly; the same text produces about 30% more tokens. See [MODELS.md](./MODELS.md) for Sonnet 5.5 details.

```python
# Before
model="claude-sonnet-4-5-20250929"

# After
model="claude-sonnet-5-5"
```

### From claude-sonnet-4-5 → claude-sonnet-4-6

```python
# Before
model="claude-sonnet-4-5-20250929"

# After
model="claude-sonnet-4-6"
```

No API changes required. Drop-in replacement with improved capabilities. (Sonnet 4.6 is still Active but is no longer the newest Sonnet; the deprecation notice names `claude-sonnet-5-5` as the replacement.)

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
