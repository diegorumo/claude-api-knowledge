# Streaming

> **Last updated:** 2026-05-30

## Overview

Streaming returns partial responses as they are generated via Server-Sent Events (SSE). This reduces perceived latency for long responses.

## Python — SDK Stream Helper

The SDK provides a high-level streaming helper:

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main():
    async with client.messages.stream(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Say hello!"}],
    ) as stream:
        async for event in stream:
            if event.type == "text":
                print(event.text, end="", flush=True)
            elif event.type == "content_block_stop":
                print()

    # Get the full accumulated message after streaming
    final = await stream.get_final_message()
    print(f"Usage: {final.usage}")

asyncio.run(main())
```

## Python — Synchronous Streaming

```python
from anthropic import Anthropic

client = Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## Python — Raw SSE Events

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
) as stream:
    for event in stream:
        print(event.type)
```

## TypeScript — SDK Stream Helper

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const stream = client.messages
  .stream({
    model: 'claude-sonnet-4-6',
    max_tokens: 1024,
    messages: [{ role: 'user', content: 'Hello!' }],
  })
  .on('text', (text) => process.stdout.write(text))
  .on('message', (msg) => console.log('\nDone:', msg.usage));

await stream.done();
const finalMessage = await stream.finalMessage();
```

## TypeScript — Raw SSE Events

```typescript
const stream = await client.messages.create({
  model: 'claude-sonnet-4-6',
  stream: true,
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }],
});

for await (const event of stream) {
  if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
    process.stdout.write(event.delta.text);
  }
}
```

## SSE Event Types

Events arrive in this sequence:

| Event Type | Description |
|-----------|-------------|
| `message_start` | Contains initial message object with `id`, `model`, empty `content` |
| `content_block_start` | Start of a content block; `index` and `content_block` with type |
| `ping` | Periodic keepalive |
| `content_block_delta` | Incremental content; `delta` contains the change |
| `content_block_stop` | End of a content block |
| `message_delta` | Contains `stop_reason`, `stop_sequence`, usage updates |
| `message_stop` | Stream complete |

## Delta Types

| Delta Type | When | Contains |
|-----------|------|--------|
| `text_delta` | Text content block | `text: string` |
| `thinking_delta` | Thinking block (extended thinking) | `thinking: string` |
| `input_json_delta` | Tool input streaming | `partial_json: string` |

## SSE Wire Format

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_xxx","type":"message","role":"assistant","content":[],"model":"claude-sonnet-4-6","stop_reason":null,"stop_sequence":null,"usage":{"input_tokens":10,"output_tokens":1}}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: ping
data: {"type":"ping"}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"!"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn","stop_sequence":null},"usage":{"output_tokens":5}}

event: message_stop
data: {"type":"message_stop"}
```

## Raw HTTP Streaming

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  --no-buffer \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "stream": true,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

## Streaming with Tool Use

When tools are involved, tool input is streamed incrementally as JSON:

```python
async with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{"name": "get_weather", "description": "...", "input_schema": {...}}],
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
) as stream:
    async for event in stream:
        if event.type == "content_block_start" and event.content_block.type == "tool_use":
            print(f"Tool called: {event.content_block.name}")
        elif event.type == "content_block_delta" and event.delta.type == "input_json_delta":
            print(event.delta.partial_json, end="")

    final = await stream.get_final_message()
    # final.stop_reason == "tool_use"
```

## TypeScript Stream Events

```typescript
const stream = client.messages.stream({ ... })
  .on('connect', () => console.log('Connected'))
  .on('text', (delta, snapshot) => process.stdout.write(delta))
  .on('contentBlock', (block) => console.log('Block:', block))
  .on('message', (msg) => console.log('Final:', msg))
  .on('error', (err) => console.error(err));

// Abort mid-stream
stream.abort();
```

## Gotchas

- Always consume the entire stream before accessing `finalMessage()` or `get_final_message()`
- `content_block_delta` events for tool inputs may arrive as fragmented JSON — accumulate `partial_json` before parsing
- Streaming does not support `top_p` + `temperature` simultaneously set to non-defaults in all models
- Extended thinking blocks stream with `thinking_delta` events — accumulate before displaying

## Related

- [Messages API](./messages-api.md)
- [Tool Use](./tool-use.md)
- [Extended Thinking](./extended-thinking.md)
