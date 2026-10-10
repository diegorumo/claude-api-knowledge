# Quick Reference

> **Last updated:** 2026-10-10  
> Common patterns for developers building with the Claude API.

## Authentication

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

```python
import anthropic
client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY
```

```typescript
import Anthropic from '@anthropic-ai/sdk';
const client = new Anthropic(); // reads ANTHROPIC_API_KEY
```

---

## Basic Message

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.content[0].text)
```

```typescript
const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }],
});
console.log(response.content[0].text);
```

---

## Current Model IDs

> Checked against platform.claude.com/docs/en/models/overview on 2026-10-10. See [MODELS.md](./MODELS.md) for prices, limits and retirement dates.

| Model | ID | Use For |
|-------|-----|--------|
| Opus 5.5 | `claude-opus-5-5` | Recommended starting point for most workloads; 1M ctx; always-on thinking |
| Fable 5.1 | `claude-fable-5-1` | Demanding reasoning, long-horizon agentic work; 2.5% cache reads; 30-day data retention required; `tool_choice any/tool` unsupported |
| Sonnet 5.5 | `claude-sonnet-5-5` | Best speed/intelligence balance; $2/$10 MTok |
| Haiku 5.5 | `claude-haiku-5-5` | Fastest; high-volume, latency-sensitive work; 1M ctx (launched Oct 7, 2026) |
| Fable 5 | `claude-fable-5` | Legacy (still available) |
| Opus 5 | `claude-opus-5` | Legacy (still available) |
| Sonnet 5 | `claude-sonnet-5` | Legacy (still available) |
| Opus 4.8 | `claude-opus-4-8` | Legacy (still available) |
| Opus 4.7 | `claude-opus-4-7` | Legacy (still available) |
| Opus 4.6 | `claude-opus-4-6` | Legacy (still available) |
| Opus 4.5 | `claude-opus-4-5-20251101` | Legacy (still available) |
| Sonnet 4.6 | `claude-sonnet-4-6` | Legacy (still available) |
| Haiku 4.5 | `claude-haiku-4-5-20251001` | Legacy (still available); 200k ctx |

> **Invitation-only:** `claude-mythos-5-1` (Project Glasswing) — same specs as Fable 5.1.  
> **Deprecated:** `claude-sonnet-4-5-20250929` — retires Nov 30, 2026; migrate to `claude-sonnet-5-5`.  
> **Retired:** `claude-opus-4-1` (Aug 5, 2026) — returns errors; migrate to a current model.

---

## Streaming

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

```typescript
const stream = client.messages.stream({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Tell me a story.' }],
}).on('text', (text) => process.stdout.write(text));
await stream.done();
```

---

## Tool Use Skeleton

```python
tools = [{
    "name": "get_weather",
    "description": "Get weather for a location",
    "input_schema": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
    },
}]

response = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
    tools=tools, messages=[{"role": "user", "content": "Weather in Paris?"}],
)

if response.stop_reason == "tool_use":
    tool = next(b for b in response.content if b.type == "tool_use")
    result = my_get_weather(tool.input["location"])
    
    final = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=1024, tools=tools,
        messages=[
            {"role": "user", "content": "Weather in Paris?"},
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": [{
                "type": "tool_result",
                "tool_use_id": tool.id,
                "content": result,
            }]},
        ],
    )
    print(final.content[0].text)
```

---

## Prompt Caching

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": large_document,
        "cache_control": {"type": "ephemeral"},  # Cache this
    }],
    messages=[{"role": "user", "content": "Summarize this."}],
)
# Check: response.usage.cache_read_input_tokens > 0 means cache hit
```

---

## Extended Thinking

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=8000,
    thinking={"type": "enabled", "budget_tokens": 3000},
    messages=[{"role": "user", "content": "Solve this complex problem..."}],
)
for block in response.content:
    if block.type == "thinking":
        print(f"<thinking>{block.thinking}</thinking>")
    elif block.type == "text":
        print(block.text)
```

---

## Web Search

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{"type": "web_search_20250305", "name": "web_search"}],
    messages=[{"role": "user", "content": "Latest Claude news?"}],
)
print(response.content[-1].text)
```

---

## Token Counting

```python
result = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Your prompt here"}],
)
print(f"Tokens: {result.input_tokens}")
```

---

## Batch API

```python
batch = client.messages.batches.create(requests=[
    {"custom_id": f"item-{i}", "params": {
        "model": "claude-sonnet-4-6", "max_tokens": 512,
        "messages": [{"role": "user", "content": item}],
    }}
    for i, item in enumerate(items)
])
# Later: client.messages.batches.results(batch.id)
```

---

## Vision

```python
import base64
response = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
    messages=[{"role": "user", "content": [
        {"type": "text", "text": "What's in this image?"},
        {"type": "image", "source": {
            "type": "base64", "media_type": "image/png",
            "data": base64.b64encode(open("image.png","rb").read()).decode(),
        }},
    ]}],
)
```

---

## Error Handling

```python
try:
    response = client.messages.create(...)
except anthropic.RateLimitError:
    time.sleep(60)  # Retry after
except anthropic.AuthenticationError:
    print("Check ANTHROPIC_API_KEY")
except anthropic.APIStatusError as e:
    print(f"Error {e.status_code}: {e.message}")
```

---

## Raw HTTP (curl)

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":1024,"messages":[{"role":"user","content":"Hello"}]}'
```

---

## Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/messages` | POST | Create message |
| `/v1/messages/count_tokens` | POST | Count tokens |
| `/v1/messages/batches` | POST | Create batch |
| `/v1/messages/batches/{id}` | GET | Get batch status |
| `/v1/messages/batches/{id}/results` | GET | Get batch results |
| `/v1/models` | GET | List models |
| `/v1/files` | POST | Upload file |
| `/v1/files/{id}` | DELETE | Delete file |

---

## Links

- [Full Documentation Index](./README.md)
- [Models](./MODELS.md)
- [Messages API](./messages-api.md)
- [Tool Use](./tool-use.md)
- [Prompt Caching](./prompt-caching.md)
- [Streaming](./streaming.md)
- [Rate Limits & Errors](./rate-limits-errors.md)
