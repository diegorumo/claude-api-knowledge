# Prompt Caching

> **Last updated:** 2026-10-10  
> **Source:** Anthropic cookbook — demonstrated 3.3x speedup on 187K-token document; platform.claude.com/docs/en/build-with-claude/cache-diagnostics; platform.claude.com/docs/en/build-with-claude/prompt-caching; platform.claude.com/docs/en/about-claude/pricing

## Overview

Prompt caching stores processed prompt prefixes so subsequent requests reuse them instead of re-processing. Benefits:

- **Latency:** 2–3x faster (cache hits skip tokenization/KV computation)
- **Cost:** Reads cost ~10% of base input price on most models; only 125% for cache writes

> **Opus 5.5 / Sonnet 5.5 cache pricing:** Cache reads (hits and refreshes) cost **5% of base input price**: $0.20/MTok on `claude-opus-5-5` and, since the **Oct 7, 2026** price cut, **$0.10/MTok** on `claude-sonnet-5-5` (was $0.20, 10%). Cache writes are unchanged. Source: platform.claude.com/docs/en/about-claude/pricing.
>
> **Haiku 5.5 cache pricing:** `claude-haiku-5-5` is priced by prompt length. Cache reads are $0.01/MTok for prompts up to 100,000 tokens and $0.05/MTok over 100,000 (10% of base input in both bands). Cache writes: $0.125 / $0.625 (5m) and $0.20 / $1 (1h) per MTok. A request's prompt length includes cache reads and writes.

> **Fable 5.1 / Mythos 5.1 cache pricing:** Cache reads on `claude-fable-5-1` and `claude-mythos-5-1` cost only **2.5% of base input price** ($0.25/MTok vs. $10/MTok full price). This is 4× cheaper than the standard 10% rate on other models.

## How It Works

Add `"cache_control": {"type": "ephemeral"}` to content blocks. The API caches everything up to that marker.

**Cache TTL:** 5 minutes (default). Extended 1-hour TTL available at 2x cache write price.

## Minimum Token Requirements

Minimums on the Claude API, Claude Platform on AWS, Google Cloud and Microsoft Foundry (source: platform.claude.com/docs/en/build-with-claude/prompt-caching#cache-limitations, checked 2026-10-10):

| Model                                                                                                                  | Minimum Cacheable Tokens |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| Fable 5.1, Mythos 5.1, Opus 5.5, Opus 5, Sonnet 5.5, Fable 5, Mythos 5, Haiku 5.5                                       | 512 tokens               |
| Opus 4.8, Sonnet 5, Sonnet 4.6, Sonnet 4.5 (deprecated); retired Opus 4.1 / Opus 4 / Sonnet 4                           | 1,024 tokens             |
| Mythos Preview, Opus 4.7                                                                                               | 2,048 tokens             |
| Opus 4.6, Opus 4.5                                                                                                     | 4,096 tokens             |
| Haiku 4.5                                                                                                              | 4,096 tokens             |
| Haiku 3.5 (retired, except on Google Cloud)                                                                            | 2,048 tokens             |

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
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const document = fs.readFileSync("large_document.txt", "utf-8");

const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  system: [
    {
      type: "text",
      text: `Analyze this document:\n\n${document}`,
      cache_control: { type: "ephemeral" },
    },
  ],
  messages: [{ role: "user", content: "What are the main themes?" }],
});

console.log("Cache created:", response.usage.cache_creation_input_tokens);
console.log("Cache read:", response.usage.cache_read_input_tokens);
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

| Token Type    | Cost                    |
| ------------- | ----------------------- |
| Cache write   | 1.25× base input price  |
| Cache read    | 0.10× base input price (0.05× on Opus 5.5 and Sonnet 5.5; 0.025× on Fable 5.1 / Mythos 5.1) |
| Regular input | 1.00× base input price  |
| Output        | 1.00× base output price |

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

## Cache Diagnostics (GA Sep 23, 2026)

Cache diagnostics tells you _why_ a cache missed by comparing a request against the previous one and reporting the first point of divergence (model, system prompt, tools or message history).

- **Out of beta on the Claude API since Sep 23, 2026.** The `cache-diagnosis-2026-04-07` header is no longer required; requests that still send it work as before.
- **Opt in per request** by including a `diagnostics` object. Turn 1: `{"previous_message_id": null}`. Later turns: `{"previous_message_id": "<id of previous response>"}`. The API stores a fingerprint (hashes and token-count estimates only, never raw prompt text) only for requests that include the object.
- **Include `diagnostics` on every turn you want to chain** (since Sep 9, 2026). A request without it stores no fingerprint, so a later turn that points `previous_message_id` at it gets `previous_message_not_found`. Sending only the old beta header is accepted but stores nothing.
- **Responses from `POST /v1/messages` always include `diagnostics`**, which is `null` when the request didn't include the object.
- **Claude API only.** Not available on Claude Platform on AWS, Amazon Bedrock, Google Cloud or Microsoft Foundry. ZDR eligible (excluding covered models).

```python
client = anthropic.Anthropic()
SYSTEM = "You are an AI assistant analyzing a large document. <document>...</document>"

# Turn 1: opt in with previous_message_id=None
r1 = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    system=SYSTEM,
    messages=[{"role": "user", "content": "Summarize section 1."}],
    diagnostics={"previous_message_id": None},
)

# Turn 2: reference the previous response id
r2 = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    system=SYSTEM,
    messages=[
        {"role": "user", "content": "Summarize section 1."},
        {"role": "assistant", "content": r1.content},
        {"role": "user", "content": "Now summarize section 2."},
    ],
    diagnostics={"previous_message_id": r1.id},
)

d = r2.diagnostics
if d is None:
    print("No divergence detected.")
elif d.cache_miss_reason is None:
    print("Comparison still pending.")
else:
    print(f"cache_miss_reason: {d.cache_miss_reason.type}")
```

```typescript
const r2 = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  system: SYSTEM,
  messages: [
    { role: "user", content: "Summarize section 1." },
    { role: "assistant", content: r1.content },
    { role: "user", content: "Now summarize section 2." },
  ],
  diagnostics: { previous_message_id: r1.id },
});

if (r2.diagnostics === null) {
  console.log("No divergence detected.");
} else if (r2.diagnostics.cache_miss_reason === null) {
  console.log("Comparison still pending.");
} else {
  console.log(`cache_miss_reason: ${r2.diagnostics.cache_miss_reason.type}`);
}
```

The official examples call `client.beta.messages`. The non-beta `Message` / `MessageCreateParams` types gained `diagnostics` in Python v1.9.0 and TypeScript v0.129.0 (Sep 28, 2026), so older SDKs need `client.beta.messages` for it. When streaming, `diagnostics` arrives on the `message_start` event.

**Response values:**

| `diagnostics`                  | Meaning                                                                                  |
| ------------------------------ | ---------------------------------------------------------------------------------------- |
| `null`                         | Request didn't opt in, `previous_message_id` was `null`, or no divergence found          |
| `{"cache_miss_reason": null}`  | Comparison still running when the response was serialized; inconclusive, check next turn |
| `{"cache_miss_reason": {...}}` | Divergence found (or no comparison possible, see types below)                            |

**`cache_miss_reason.type` values** (earliest divergence only):

| Type                         | Meaning / fix                                                                                                                                                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_changed`              | Model differs from previous request. Hold the model constant within a cached conversation                                                                                                                        |
| `system_changed`             | `system` differs (often an interpolated timestamp/ID). Make it byte-stable; move dynamic data into the first user message                                                                                        |
| `tools_changed`              | Tools added, removed, reordered, or schemas serialized non-deterministically. Send a fixed, deterministically serialized list                                                                                    |
| `messages_changed`           | An earlier message was edited, reordered or removed. Treat history as append-only; echo content back verbatim                                                                                                    |
| `previous_message_not_found` | No fingerprint for that ID (previous request didn't opt in, different workspace, or too old). Not evidence of a change                                                                                           |
| `unavailable`                | No diagnosis possible, e.g. `tool_choice`, `thinking`, `context_management`, `output_config`, `output_format` or the set of `anthropic-beta` headers changed, or the divergence is beyond the comparison horizon |

The four `*_changed` types also carry `cache_missed_input_tokens`, a rough estimate (from byte lengths) of input tokens after the divergence point. Use it as a magnitude indicator, not a billing number.

**Reading with usage:** `diagnostics: null` + high `cache_read_input_tokens` = working. `null` + low reads = requests matched but the cache entry expired (shorten gaps or use 1-hour TTL). A `*_changed` type + low reads = your request changed; fix it.

**Limitations:** fingerprints expire after a short period; both requests must be in the same organization and workspace (compare the `anthropic-workspace-id` response header); best-effort, never blocks or fails the request.

## Best Practices

1. **Start with automatic caching** — add one `cache_control` to your system prompt or document
2. **Place breakpoints strategically** — cache the static parts (system, docs, tools); leave dynamic parts uncached
3. **Order matters** — cache_control applies to everything up to that marker; put static content first
4. **Measure cache performance** — monitor `cache_read_input_tokens` to verify hits; use [cache diagnostics](#cache-diagnostics-ga-sep-23-2026) to find the cause of misses
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
