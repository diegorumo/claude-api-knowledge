# Token Counting

> **Last updated:** 2026-05-30

## Overview

Count tokens in a request before sending it. Useful for estimating cost, checking if input fits the context window, or managing prompt length dynamically.

**Endpoint:** `POST /v1/messages/count_tokens`

## Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

result = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    messages=[
        {"role": "user", "content": "Hello, Claude!"}
    ],
)
print(f"Input tokens: {result.input_tokens}")
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const result = await client.messages.countTokens({
  model: 'claude-sonnet-4-6',
  messages: [{ role: 'user', content: 'Hello, Claude!' }],
});
console.log(`Input tokens: ${result.input_tokens}`);
```

## With System Prompt and Tools

```python
result = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    system="You are a helpful assistant.",
    tools=[
        {
            "name": "get_weather",
            "description": "Get current weather",
            "input_schema": {
                "type": "object",
                "properties": {"location": {"type": "string"}},
            },
        }
    ],
    messages=[
        {"role": "user", "content": "What's the weather in Paris?"}
    ],
)
print(f"Total input tokens: {result.input_tokens}")
```

## Response Format

```json
{"input_tokens": 42}
```

Note: `count_tokens` only returns `input_tokens` — output tokens are unknown before generation.

## Raw HTTP

```bash
curl https://api.anthropic.com/v1/messages/count_tokens \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Context Window Management

```python
MAX_CONTEXT = 180_000  # Leave headroom in 200K window

def trim_messages(messages, model, max_tokens=MAX_CONTEXT):
    while len(messages) > 1:
        result = client.messages.count_tokens(model=model, messages=messages)
        if result.input_tokens <= max_tokens:
            return messages
        messages = messages[2:]  # Remove oldest user+assistant pair
    return messages
```

## Gotchas

- Token counts may differ slightly from actual billed tokens (±1–2%) due to formatting
- Tool definitions and system prompts add tokens beyond visible text — always count the full request
- Images add variable tokens based on dimensions; use `count_tokens` with the actual image for accuracy
- Cache control markers themselves add a small number of overhead tokens

## Related

- [Messages API](./messages-api.md)
- [Prompt Caching](./prompt-caching.md)
- [Vision](./vision.md)
- [Models](./MODELS.md)
