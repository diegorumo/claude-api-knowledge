# Web Search Tool

> **Last updated:** 2026-06-08

## Overview

The web search and web fetch tools are built-in server-side tools that let Claude search the internet or fetch URLs. Unlike custom tools, you don't need to execute them — the API handles them internally.

**Web Search tool name:** `web_search`  
**Latest web search type:** `web_search_20260209` (also: `web_search_20250305`)

**Web Fetch tool name:** `web_fetch`  
**Latest web fetch type:** `web_fetch_20260309` (also: `web_fetch_20260209`)

## Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What are the latest Claude model releases?"}],
    tools=[
        {
            "type": "web_search_20260209",  # latest; use web_search_20250305 for older compat
            "name": "web_search",
        }
    ],
)

for block in message.content:
    if block.type == "text":
        print(block.text)

if message.usage.server_tool_use:
    print(f"Web searches: {message.usage.server_tool_use.web_search_requests}")
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const message = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'What is the current price of Bitcoin?' }],
  tools: [{ type: 'web_search_20260209', name: 'web_search' }],
});

for (const block of message.content) {
  if (block.type === 'text') console.log(block.text);
}
```

## Web Fetch Tool

The web fetch tool lets Claude retrieve and read the contents of specific URLs:

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[{"role": "user", "content": "Fetch and summarize https://example.com"}],
    tools=[{"type": "web_fetch_20260309", "name": "web_fetch"}],
)
```

---

## Streaming with Web Search

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What happened in tech news today?"}],
    tools=[{"type": "web_search_20260209", "name": "web_search"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

## Usage Tracking

Web search requests are tracked separately:

```json
{
  "usage": {
    "input_tokens": 25,
    "output_tokens": 350,
    "server_tool_use": {
      "web_search_requests": 2
    }
  }
}
```

Web search requests are billed per search — check https://www.anthropic.com/pricing for current rates.

## How It Works

1. You include `web_search` in your tools list
2. Claude decides when to search based on the query
3. The API executes the search server-side
4. Claude receives results and incorporates them into its response

You **do not** need to handle tool results for `web_search` — unlike custom tools, results are fed back automatically.

## Best Practices

- Use web search for time-sensitive information (news, prices, current events)
- Claude cites sources automatically — good for research applications
- For known-stable information, skip web search to save cost
- Web search adds latency; for real-time applications, set appropriate timeouts

## Gotchas

- Web search incurs additional cost per search request
- Results quality depends on the search query Claude generates
- Claude may search multiple times for complex questions

## Related

- [Tool Use](./tool-use.md)
- [Messages API](./messages-api.md)
- [MCP Integration](./mcp.md)
