# Extended Thinking

> **Last updated:** 2026-05-30

## Overview

Extended thinking gives Claude internal reasoning space before generating its response. The model produces a `thinking` content block with its reasoning process, then a `text` block with the final answer.

**Available on:** Claude Opus 4.8, Claude Sonnet 4.6 (not Haiku)

## Enabling Extended Thinking

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8000,  # Must be > budget_tokens
    thinking={
        "type": "enabled",
        "budget_tokens": 5000,  # How many tokens Claude can spend thinking
    },
    messages=[{"role": "user", "content": "What is 27 * 453? Think carefully."}],
)

for block in response.content:
    if block.type == "thinking":
        print(f"<thinking>\n{block.thinking}\n</thinking>")
    elif block.type == "text":
        print(f"Answer: {block.text}")
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const message = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 8000,
  thinking: { type: 'enabled', budget_tokens: 5000 },
  messages: [{ role: 'user', content: 'Solve this step by step: 127 * 89' }],
});

for (const block of message.content) {
  if (block.type === 'thinking') {
    console.log('Thinking:', block.thinking);
  } else if (block.type === 'text') {
    console.log('Answer:', block.text);
  }
}
```

## Thinking Modes

| Mode | Description |
|------|-------------|
| `{"type": "enabled", "budget_tokens": N}` | Explicit budget; Claude uses up to N tokens |
| `{"type": "adaptive"}` | Claude decides how much to think (no explicit budget) |
| `{"type": "disabled"}` | No thinking (default) |

## Budget Tokens

- `budget_tokens` must be less than `max_tokens`
- Typical values: 1,000–10,000 for simple tasks; 10,000–32,000 for complex reasoning
- Claude may use fewer tokens than the budget if the problem is simple
- Higher budget = more thorough reasoning but more latency and cost

```python
# For complex math / multi-step reasoning
thinking={"type": "enabled", "budget_tokens": 10000}

# For simple tasks that benefit from brief reflection
thinking={"type": "enabled", "budget_tokens": 1024}

# Let Claude decide
thinking={"type": "adaptive"}
```

## Streaming with Thinking

Thinking blocks stream with `thinking_delta` events:

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main():
    async with client.messages.stream(
        model="claude-sonnet-4-6",
        max_tokens=8000,
        thinking={"type": "enabled", "budget_tokens": 3000},
        messages=[{"role": "user", "content": "Explain quantum entanglement."}],
    ) as stream:
        async for event in stream:
            if event.type == "content_block_start":
                if event.content_block.type == "thinking":
                    print("<thinking>", end="", flush=True)
                elif event.content_block.type == "text":
                    print("\n<answer>", end="", flush=True)
            elif event.type == "content_block_delta":
                if event.delta.type == "thinking_delta":
                    print(event.delta.thinking, end="", flush=True)
                elif event.delta.type == "text_delta":
                    print(event.delta.text, end="", flush=True)

asyncio.run(main())
```

```typescript
const stream = client.messages.stream({
  model: 'claude-sonnet-4-6',
  max_tokens: 8000,
  thinking: { type: 'enabled', budget_tokens: 3000 },
  messages: [{ role: 'user', content: 'Analyze this problem...' }],
});

for await (const event of stream) {
  if (event.type === 'content_block_delta') {
    if (event.delta.type === 'thinking_delta') {
      process.stdout.write(event.delta.thinking);
    } else if (event.delta.type === 'text_delta') {
      process.stdout.write(event.delta.text);
    }
  }
}
```

## Response Content Block Structure

```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "Let me work through this step by step...\n27 × 453\n= 27 × 400 + 27 × 53\n..."
    },
    {
      "type": "text",
      "text": "27 × 453 = 12,231"
    }
  ]
}
```

## Multi-Turn with Thinking

Pass thinking blocks back in subsequent turns for continuity:

```python
response1 = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    thinking={"type": "enabled", "budget_tokens": 2000},
    messages=[{"role": "user", "content": "Analyze this dataset..."}],
)

# Include the thinking block in the next turn
messages = [
    {"role": "user", "content": "Analyze this dataset..."},
    {"role": "assistant", "content": response1.content},  # Includes thinking block
    {"role": "user", "content": "Now summarize your findings."},
]

response2 = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=4096,
    thinking={"type": "enabled", "budget_tokens": 2000},
    messages=messages,
)
```

## When to Use Extended Thinking

**Good use cases:**
- Complex multi-step math
- Logical reasoning and puzzles
- Code analysis and debugging
- Research synthesis across many facts
- Planning and strategy tasks

**Less beneficial:**
- Simple factual questions
- Creative writing (thinking adds cost without benefit)
- Tasks where latency matters more than accuracy

## Gotchas

- `max_tokens` must exceed `budget_tokens` — the model needs room for the actual response
- Thinking blocks are visible to users if you surface them; consider whether to display them
- Thinking tokens are billed at the output token rate
- Not available on Haiku models

## Related

- [Messages API](./messages-api.md)
- [Models](./MODELS.md)
- [Streaming](./streaming.md)
