# Quick Reference

> **Last updated:** 2026-07-06  
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

| Model | ID | Use For |
|-------|-----|--------|
| Fable 5 | `claude-fable-5` | New top-tier (June 2026) |
| Mythos 5 | `claude-mythos-5` | New model family (June 2026) |
| Opus 4.8 | `claude-opus-4-8` | Highest Opus capability |
| Opus 4.7 | `claude-opus-4-7` | High capability |
| Opus 4.6 | `claude-opus-4-6` | Recommended Opus |
| Sonnet 5 | `claude-sonnet-5` | New Sonnet generation (June 2026) |
| Sonnet 4.6 | `claude-sonnet-4-6` | Balanced (default choice) |
| Haiku 4.5 | `claude-haiku-4-5-20251001` | Fastest / cheapest |

> **Deprecated:** `claude-opus-4-1` — migrate to `claude-opus-4-6`.  
> **New:** `claude-fable-5` and `claude-mythos-5` support server-side fallbacks on refusal.  
> **New:** `claude-sonnet-5` added June 2026 — check Anthropic docs for full capability details.

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
