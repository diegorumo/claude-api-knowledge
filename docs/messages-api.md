# Messages API

> **Last updated:** 2026-06-01

## Overview

The Messages API is the primary interface for interacting with Claude. It accepts a list of messages and returns a model-generated response.

**Endpoint:** `POST /v1/messages`

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Model ID (e.g. `claude-sonnet-4-6`) |
| `max_tokens` | integer | Yes | Maximum tokens in response (1–model max) |
| `messages` | array | Yes | Conversation history (alternating user/assistant) |
| `system` | string or array | No | System prompt |
| `tools` | array | No | Tool definitions for function calling |
| `tool_choice` | object | No | Control tool selection |
| `thinking` | object | No | Enable extended thinking |
| `stream` | boolean | No | Enable SSE streaming |
| `temperature` | float | No | Randomness 0–1 (default 1) |
| `top_p` | float | No | Nucleus sampling 0–1 |
| `top_k` | integer | No | Top-k sampling |
| `stop_sequences` | array | No | Custom stop strings |
| `metadata` | object | No | User ID for abuse detection |
| `container` | string | No | Container ID for request reuse (execution environment) |
| `inference_geo` | string | No | Geographic region hint for inference routing |
| `service_tier` | string | No | `"auto"` or `"standard_only"` — capacity tier selection |
| `output_config` | object | No | Output format/effort configuration (see below) |

## output_config Parameter

Control output quality/effort and structured JSON format:

```python
# Set effort level (affects quality vs latency tradeoff)
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    output_config={"effort": "high"},  # "low", "medium", "high", "xhigh", "max"
    messages=[{"role": "user", "content": "Analyze this complex problem..."}],
)

# Structured JSON output with schema
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"]
            }
        }
    },
    messages=[{"role": "user", "content": "Extract: John is 30 years old."}],
)
```

## Message Format

Each message has a `role` (`"user"`, `"assistant"`, or `"system"` for mid-conversation) and `content` (string or array of content blocks).

```python
messages = [
    {"role": "user", "content": "Hello!"},
    {"role": "assistant", "content": "Hi! How can I help?"},
    {"role": "user", "content": "Tell me a joke."},
]
```

### Mid-Conversation System Blocks (SDK v0.105.0+)

You can inject system-level instructions at any point in a conversation using `role: "system"` with a `MidConversationSystemBlock`:

```python
messages = [
    {"role": "user", "content": "Translate the following text."},
    {"role": "assistant", "content": "Sure, what would you like me to translate?"},
    # Inject new instructions mid-conversation
    {
        "role": "system",
        "content": [
            {
                "type": "mid_conv_system",
                "content": [{"type": "text", "text": "Always respond in formal English only."}],
            }
        ],
    },
    {"role": "user", "content": "Translate: Bonjour le monde"},
]
```

The `mid_conv_system` block can also carry `cache_control` for caching those instructions.

## Content Block Types

```json
// Text
{"type": "text", "text": "Hello world"}

// Image (base64)
{
  "type": "image",
  "source": {
    "type": "base64",
    "media_type": "image/png",
    "data": "<base64-encoded-data>"
  }
}

// Image (URL)
{
  "type": "image",
  "source": {
    "type": "url",
    "url": "https://example.com/image.png"
  }
}

// Document (PDF via base64)
{
  "type": "document",
  "source": {
    "type": "base64",
    "media_type": "application/pdf",
    "data": "<base64-encoded-pdf>"
  }
}

// Tool result
{
  "type": "tool_result",
  "tool_use_id": "toolu_xxx",
  "content": [{"type": "text", "text": "Result here"}]
}
```

## Basic Example

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Hello, Claude"}
    ],
)
print(message.content[0].text)
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const message = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello, Claude' }],
});
console.log(message.content[0].text);
```

## With System Prompt

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a helpful assistant that responds only in haiku.",
    messages=[{"role": "user", "content": "What is the weather like?"}],
)
```

## Multi-Turn Conversation

```python
messages = []

# Turn 1
messages.append({"role": "user", "content": "Hello!"})
response = client.messages.create(model="claude-sonnet-4-6", max_tokens=1024, messages=messages)

# Append assistant response
messages.append({"role": response.role, "content": response.content})

# Turn 2
messages.append({"role": "user", "content": "How are you?"})
response2 = client.messages.create(model="claude-sonnet-4-6", max_tokens=1024, messages=messages)
```

## Response Format

```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ],
  "model": "claude-sonnet-4-6",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "stop_details": null,
  "usage": {
    "input_tokens": 10,
    "output_tokens": 25,
    "output_tokens_details": {
      "thinking_tokens": 0
    },
    "cache_creation_input_tokens": 0,
    "cache_creation": {
      "ephemeral_5m_input_tokens": 0,
      "ephemeral_1h_input_tokens": 0
    },
    "cache_read_input_tokens": 0,
    "service_tier": "standard"
  }
}
```

### stop_details

When a response is refused, `stop_details` is populated:

```json
{
  "stop_reason": "end_turn",
  "stop_details": {
    "type": "refusal",
    "category": "cyber",
    "explanation": "This request was refused due to policy."
  }
}
```

| Field | Values | Description |
|-------|--------|-------------|
| `type` | `"refusal"` | Always `"refusal"` when present |
| `category` | `"cyber"`, `"bio"`, `null` | Policy category that triggered refusal |
| `explanation` | string or null | Human-readable reason (may change over time) |

### output_tokens_details

When extended thinking is enabled, `output_tokens_details.thinking_tokens` shows how many output tokens were internal reasoning:

```python
usage = response.usage
if usage.output_tokens_details:
    print(f"Thinking tokens: {usage.output_tokens_details.thinking_tokens}")
    print(f"Response tokens: {usage.output_tokens - usage.output_tokens_details.thinking_tokens}")
```

## Stop Reasons

| Stop Reason | Meaning |
|-------------|--------|
| `end_turn` | Natural completion |
| `max_tokens` | Hit `max_tokens` limit |
| `tool_use` | Claude is requesting a tool call |
| `stop_sequence` | Hit a custom stop sequence |

## System Prompt as Array (with caching)

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are an expert assistant.",
            "cache_control": {"type": "ephemeral"},
        }
    ],
    messages=[{"role": "user", "content": "Explain quantum computing."}],
)
```

## Temperature and Sampling

```python
# More creative responses
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    temperature=0.9,
    messages=[{"role": "user", "content": "Write a poem."}],
)

# More deterministic
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    temperature=0.0,
    messages=[{"role": "user", "content": "What is 2+2?"}],
)
```

## Stop Sequences

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    stop_sequences=["END", "STOP"],
    messages=[{"role": "user", "content": "List 5 items."}],
)
```

## Metadata

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    metadata={"user_id": "user_12345"},
    messages=[{"role": "user", "content": "Hello!"}],
)
```

## Raw HTTP

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Related

- [Streaming](./streaming.md)
- [Tool Use](./tool-use.md)
- [Vision](./vision.md)
- [Prompt Caching](./prompt-caching.md)
- [Extended Thinking](./extended-thinking.md)
- [Token Counting](./token-counting.md)
