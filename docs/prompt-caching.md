# Prompt Caching

> **Last updated:** 2026-05-30  
> **Source:** Anthropic cookbook — demonstrated 3.3x speedup on 187K-token document

## Overview

Prompt caching stores processed prompt prefixes so subsequent requests reuse them instead of re-processing. Benefits:
- **Latency:** 2–3x faster (cache hits skip tokenization/KV computation)
- **Cost:** Reads cost ~10% of base input price; only 125% for cache writes

## How It Works

Add `"cache_control": {"type": "ephemeral"}` to content blocks. The API caches everything up to that marker.

**Cache TTL:** 5 minutes (default). Extended 1-hour TTL available at 2x cache write price.

## Minimum Token Requirements

| Model Family | Minimum Cacheable Tokens |
|-------------|---------------------------|
| Sonnet models | 1,024 tokens |
| Opus / Haiku models | 4,096 tokens |

Content below the minimum is never cached (no error — just no cache).

## Automatic Caching (Recommended)

Add `cache_control` to the top-level content block. The API manages cache breakpoints automatically:

```python
import anthropic

client = anthropic.Anthropic()

# Large document loaded once, then reused across requests
with open("large_document.txt") as f:
    document = f.read()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are an expert analyst. Analyze the following document:\n\n" + document,
            "cache_control": {"type": "ephemeral"},
        }
    ],
    messages=[{"role": "user", "content": "Summarize the key points."}],
)

print(f"Cache created: {response.usage.cache_creation_input_tokens}")
print(f"Cache read: {response.usage.cache_read_input_tokens}")
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const document = fs.readFileSync('large_document.txt', 'utf-8');

const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  system: [
    {
      type: 'text',
      text: `Analyze this document:\n\n${document}`,
      cache_control: { type: 'ephemeral' },
    },
  ],
  messages: [{ role: 'user', content: 'What are the main themes?' }],
});

console.log('Cache created:', response.usage.cache_creation_input_tokens);
console.log('Cache read:', response.usage.cache_read_input_tokens);
```

## Explicit Cache Breakpoints

Place `cache_control` on specific blocks for granular control. Max 4 breakpoints per request.

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are a coding assistant.",
            "cache_control": {"type": "ephemeral"},  # Cache system prompt separately
        }
    ],
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": codebase_context,  # Large context cached here
                    "cache_control": {"type": "ephemeral"},
                },
                {
                    "type": "text",
                    "text": "What does the authentication module do?",
                    # No cache_control — this changes each request
                },
            ],
        }
    ],
)
```

## Multi-Turn Conversation Caching

In multi-turn conversations, cache the growing context to minimize repeated processing:

```python
messages = []
system = [{"type": "text", "text": large_context, "cache_control": {"type": "ephemeral"}}]

while True:
    user_input = input("You: ")
    messages.append({"role": "user", "content": user_input})
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=system,
        messages=messages,
    )
    
    assistant_msg = response.content[0].text
    messages.append({"role": "assistant", "content": assistant_msg})
    print(f"Claude: {assistant_msg}")
    # After turn 1: nearly 100% of input tokens served from cache
```

## Usage Tracking

The response `usage` object shows cache performance:

```python
usage = response.usage
print(f"Input tokens (non-cached): {usage.input_tokens}")
print(f"Cache write tokens: {usage.cache_creation_input_tokens}")
print(f"Cache read tokens: {usage.cache_read_input_tokens}")
print(f"Output tokens: {usage.output_tokens}")
```

## Pricing Structure

| Token Type | Cost |
|-----------|------|
| Cache write | 1.25× base input price |
| Cache read | 0.10× base input price |
| Regular input | 1.00× base input price |
| Output | 1.00× base output price |

**Break-even point:** A single cache hit covering the same token count as the write pays for the write cost (0.10 vs 1.25). After ~2 cache hits, you save money on every additional request.

## Extended TTL (1-Hour Cache)

```python
# 1-hour TTL (costs 2x normal cache write price)
"cache_control": {"type": "ephemeral", "ttl": 3600}
```

## What Can Be Cached

- System prompts ✅
- Tool definitions ✅  
- Long documents in user messages ✅
- Conversation history ✅
- Images (counted by token equivalent) ✅

## What Cannot Be Cached

- The actual user query (last content block — it changes every request)
- Content below minimum token threshold
- More than 4 breakpoints in a single request

## Best Practices

1. **Start with automatic caching** — add one `cache_control` to your system prompt or document
2. **Place breakpoints strategically** — cache the static parts (system, docs, tools); leave dynamic parts uncached
3. **Order matters** — cache_control applies to everything up to that marker; put static content first
4. **Measure cache performance** — monitor `cache_read_input_tokens` to verify hits
5. **Warm the cache** — first call writes; subsequent calls within TTL read

## Gotchas

- Cache is per-account and per-model; a cache miss on Sonnet doesn't help Haiku
- Changing **any** token before the cache_control marker invalidates the cache
- `cache_creation_input_tokens` being 0 on a re-request means you got a cache hit
- Tool definitions count toward minimum cacheable tokens

## Related

- [Messages API](./messages-api.md)
- [Models](./MODELS.md)
- [Batch API](./batch-api.md)
