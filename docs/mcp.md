# MCP (Model Context Protocol) Integration

> **Last updated:** 2026-05-30  
> **Status:** Beta (`anthropic-beta: mcp-client-2025-04-04`)

## Overview

MCP lets Claude connect to external services as MCP servers. Instead of writing custom tool implementations, point Claude at a running MCP server and it uses that server's tools directly.

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

const stream = anthropic.beta.messages.stream(
  {
    model: 'claude-sonnet-4-6',
    max_tokens: 1000,
    mcp_servers: [
      {
        type: 'url',
        url: 'http://example-server.modelcontextprotocol.io/sse',
        name: 'my-server',
        authorization_token: 'YOUR_TOKEN',
        tool_configuration: {
          enabled: true,
          allowed_tools: ['echo', 'add'],
        },
      },
    ],
    messages: [{ role: 'user', content: 'Use the server to calculate 1+2' }],
  },
  {
    headers: { 'anthropic-beta': 'mcp-client-2025-04-04' },
  },
);

for await (const event of stream) {
  if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
    process.stdout.write(event.delta.text);
  }
}
```

## Python Example

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1000,
    mcp_servers=[
        {
            "type": "url",
            "url": "http://your-mcp-server.example.com/sse",
            "name": "my-tools",
            "authorization_token": "YOUR_TOKEN",
            "tool_configuration": {
                "enabled": True,
                "allowed_tools": ["tool1", "tool2"],
            },
        }
    ],
    messages=[{"role": "user", "content": "Use tool1 to get me some data"}],
    extra_headers={"anthropic-beta": "mcp-client-2025-04-04"},
)
```

## MCP Server Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `"url"` for remote servers |
| `url` | Yes | SSE endpoint URL of the MCP server |
| `name` | Yes | Identifier for this server |
| `authorization_token` | No | Auth token sent to MCP server |
| `tool_configuration` | No | Restrict which tools Claude can use |

## Multiple MCP Servers

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2000,
    mcp_servers=[
        {"type": "url", "url": "https://github-mcp.example.com/sse", "name": "github", "authorization_token": github_token},
        {"type": "url", "url": "https://slack-mcp.example.com/sse", "name": "slack", "authorization_token": slack_token},
    ],
    messages=[{"role": "user", "content": "Check my GitHub PRs and post a summary to Slack"}],
    extra_headers={"anthropic-beta": "mcp-client-2025-04-04"},
)
```

## Tool Allowlisting

```python
mcp_servers=[
    {
        "type": "url",
        "url": "https://broad-mcp-server.com/sse",
        "name": "limited",
        "tool_configuration": {
            "enabled": True,
            "allowed_tools": ["read_file", "list_directory"],
            # write_file blocked for safety
        },
    }
]
```

## Differences from Custom Tools

| Feature | Custom Tools | MCP Tools |
|---------|-------------|----------|
| Tool execution | Your code | MCP server |
| Tool discovery | Defined in request | Auto-discovered from server |
| Auth management | N/A | Token sent to server |
| State persistence | Stateless | Server can maintain state |

## Gotchas

- Requires `anthropic-beta: mcp-client-2025-04-04` header
- MCP servers must be publicly accessible (HTTPS for production)
- SSE connections have timeouts; long-running tool calls may fail

## Related

- [Tool Use](./tool-use.md)
- [Managed Agents](./managed-agents.md)
