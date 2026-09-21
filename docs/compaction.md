# Conversation Compaction

**Summary:** Automatically or on-demand summarize older conversation context to extend effective context length in long-running sessions without managing summaries client-side.

**Status:** Beta  
**Beta headers:** `compact-2026-01-12` (automatic) | `compact-2026-09-04` (on-demand)

---

## Overview

Compaction addresses the context window limit in long conversations by summarizing earlier turns, replacing them with a signed `compaction` block. The API automatically drops all content blocks prior to a `compaction` block when it appears in `messages`, so you append responses as normal and let the API handle trimming.

Two modes:

| Mode | Beta Header | Trigger |
|------|------------|---------|
| Automatic | `compact-2026-01-12` | Token count threshold in `context_management.edits` |
| On-demand | `compact-2026-09-04` | `"compact": true` in the request body |

**Supported models:** `claude-fable-5-1`, `claude-mythos-5-1`, `claude-fable-5`, `claude-mythos-5`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-5`, `claude-sonnet-4-6`

---

## Automatic Compaction (`compact-2026-01-12`)

Compaction fires when input tokens reach the configured `trigger.value`. Minimum trigger: 50,000 tokens.

### Basic setup

```python
import anthropic

client = anthropic.Anthropic()
messages = [{"role": "user", "content": "Help me build a website"}]

response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112"}]},
)

# Append the full response (including any compaction block) to continue
messages.append({"role": "assistant", "content": response.content})
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const messages: Anthropic.Beta.BetaMessageParam[] = [
  { role: "user", content: "Help me build a website" }
];

const response = await client.beta.messages.create({
  betas: ["compact-2026-01-12"],
  model: "claude-opus-5",
  max_tokens: 4096,
  messages,
  context_management: { edits: [{ type: "compact_20260112" }] },
});

messages.push({ role: "assistant", content: response.content });
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | string | required | `"compact_20260112"` |
| `trigger.type` | string | `"input_tokens"` | Only supported value |
| `trigger.value` | integer | `150000` | Tokens before compaction fires; min 50,000 |
| `pause_after_compaction` | boolean | `false` | Return early with `stop_reason: "compaction"` after generating summary |
| `instructions` | string | `null` | Custom summarization prompt; **replaces** (does not supplement) the default |

### Custom trigger threshold

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={
        "edits": [{
            "type": "compact_20260112",
            "trigger": {"type": "input_tokens", "value": 100_000},
        }]
    },
)
```

### Custom summarization instructions

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={
        "edits": [{
            "type": "compact_20260112",
            "instructions": "Focus on preserving code snippets, variable names, and technical decisions.",
        }]
    },
)
```

### Pausing after compaction

`pause_after_compaction: true` stops after the summary and returns `stop_reason: "compaction"`, letting you inspect or modify before continuing.

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={
        "edits": [{"type": "compact_20260112", "pause_after_compaction": True}]
    },
)

if response.stop_reason == "compaction":
    messages.append({"role": "assistant", "content": response.content})
    # Continue with normal request
    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,
        context_management={"edits": [{"type": "compact_20260112"}]},
    )
```

---

## On-Demand Compaction (`compact-2026-09-04`)

Request a summary at any point by adding `"compact": true` to the request. The API runs the summary in the background and returns a signed `compaction` block you can swap into `messages` on later requests.

```python
response = client.beta.messages.create(
    betas=["compact-2026-09-04"],
    model="claude-opus-5",
    max_tokens=4096,
    compact=True,
    messages=messages,
)
```

Recent turns can be preserved word-for-word after the summary. Thinking blocks in preserved turns remain valid on models that support preserved thinking.

---

## Response format

When compaction fires, the response `content` array includes a `compaction` block before any new text:

```json
{
  "content": [
    {
      "type": "compaction",
      "content": "Summary of the conversation: The user requested help building a web scraper..."
    },
    {
      "type": "text",
      "text": "Based on our conversation so far..."
    }
  ],
  "stop_reason": "end_turn"
}
```

Always append the full `response.content` (including `compaction` blocks) when building the next turn. The API automatically drops all content blocks prior to a `compaction` block on the next request.

---

## Streaming

Compaction blocks stream as a single `content_block_delta` with `delta.type == "compaction_delta"`:

```python
with client.beta.messages.stream(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112"}]},
) as stream:
    for event in stream:
        match event.type:
            case "content_block_start":
                if event.content_block.type == "compaction":
                    print("Compaction in progress...")
            case "content_block_delta":
                if event.delta.type == "compaction_delta":
                    print(f"Compaction complete: {len(event.delta.content)} chars")
```

---

## Billing

Compaction requires an additional sampling iteration billed separately. The top-level `usage.input_tokens` and `usage.output_tokens` **do not** include compaction iteration usage. Sum all entries in `usage.iterations`:

```json
{
  "usage": {
    "input_tokens": 23000,
    "output_tokens": 1000,
    "iterations": [
      {
        "type": "compaction",
        "input_tokens": 180000,
        "output_tokens": 3500
      },
      {
        "type": "message",
        "input_tokens": 23000,
        "output_tokens": 1000
      }
    ]
  }
}
```

**Total billed tokens = sum of all `iterations[*].input_tokens` + sum of all `iterations[*].output_tokens`**

---

## Prompt caching with compaction

Cache the `compaction` block to save on re-reads in long sessions:

```python
# Add cache_control to the compaction block when building the next turn
messages.append({
    "role": "assistant",
    "content": [
        {
            "type": "compaction",
            "content": "[summary text]",
            "cache_control": {"type": "ephemeral"}
        },
        {"type": "text", "text": "Based on our conversation..."}
    ]
})
```

Cache system prompts separately to maximize hits:

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    max_tokens=4096,
    system=[{
        "type": "text",
        "text": "You are a helpful coding assistant...",
        "cache_control": {"type": "ephemeral"},
    }],
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112"}]},
)
```

---

## Token counting

Token counting reflects existing `compaction` blocks but does not trigger new ones:

```python
count = client.beta.messages.count_tokens(
    betas=["compact-2026-01-12"],
    model="claude-opus-5",
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112"}]},
)
print(f"Current tokens: {count.input_tokens}")
print(f"Original tokens: {count.context_management.original_input_tokens}")
```

---

## Limitations and gotchas

- **Minimum trigger value:** 50,000 tokens.
- **Server tools:** Compaction is checked at the start of each sampling iteration, so multiple compactions can occur in a single request with server tools.
- **Fable 5.1 / Mythos 5.1 thinking blocks:** Thinking blocks from before a `compaction` block are not carried forward. When re-inserting turns, remove `thinking` and `redacted_thinking` blocks, or send `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` with the `thinking-binding-controls-2026-08-01` beta header.
- **Custom instructions replace defaults:** Providing `instructions` completely replaces the default summarization prompt, not supplements it.

---

## Complete example: long-running chat loop

```python
client = anthropic.Anthropic()
messages: list[dict] = []

def chat(user_message: str) -> str:
    messages.append({"role": "user", "content": user_message})
    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="claude-opus-5",
        max_tokens=4096,
        messages=messages,
        context_management={
            "edits": [{"type": "compact_20260112", "trigger": {"type": "input_tokens", "value": 100_000}}]
        },
    )
    messages.append({"role": "assistant", "content": response.content})
    return next(b.text for b in response.content if b.type == "text")

print(chat("Help me build a Python web scraper"))
print(chat("Add support for JavaScript-rendered pages"))
print(chat("Now add rate limiting and error handling"))
```

---

## Related

- [Prompt Caching](./prompt-caching.md)
- [Extended Thinking](./extended-thinking.md)
- [Token Counting](./token-counting.md)
- [Streaming](./streaming.md)
