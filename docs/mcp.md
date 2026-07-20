# MCP (Model Context Protocol) Integration

> **Last updated:** 2026-07-20  
> **Status:** Beta (`anthropic-beta: mcp-client-2025-04-04` for client; `mcp-tunnels-2026-06-22` for tunnels)

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

---

## MCP Tunnels (Python v0.117.0 / TypeScript v0.112.0 — Beta)

> **Beta header:** `anthropic-beta: mcp-tunnels-2026-06-22`

MCP Tunnels let you expose a **locally-running MCP server** to Claude's infrastructure over a secure TLS tunnel without opening public firewall ports. Anthropic provisions a stable hostname (`domain`) for each tunnel; you run a connector process on your side using the `tunnel_token`.

### How it works

1. **Create a tunnel** — Anthropic allocates a unique hostname (`domain`) prefixed `tnl_`.
2. **Register a CA certificate** — Upload the CA cert whose chain your MCP server presents. Anthropic verifies the server's TLS cert against it.
3. **Start the connector** — Run the connector process with the `tunnel_token`; it bridges Anthropic's infra to your local server.
4. **Point `mcp_servers` at the tunnel domain** — Use `type: "url"` with the tunnel's domain as the hostname in your Messages API call.
5. **Rotate or archive tokens as needed** — Rotate the token to invalidate old connectors; archive a tunnel irreversibly when done.

### Tunnels Endpoints

| Method | HTTP | Description |
|--------|------|-------------|
| `create` | `POST /v1/tunnels?beta=true` | Provision a new tunnel (not idempotent) |
| `retrieve` | `GET /v1/tunnels/{id}?beta=true` | Fetch tunnel by ID |
| `list` | `GET /v1/tunnels?beta=true` | List tunnels (newest-first, archived excluded by default) |
| `archive` | `POST /v1/tunnels/{id}/archive?beta=true` | Irreversibly archive (retires hostname, invalidates token) |
| `reveal_token` | `POST /v1/tunnels/{id}/reveal_token?beta=true` | Fetch connector token live (Anthropic does not store it) |
| `rotate_token` | `POST /v1/tunnels/{id}/rotate_token?beta=true` | Issue new token; does not sever established connections |

### Certificates Endpoints

Each tunnel supports **at most 2 non-archived certificates**. Archiving the last active cert causes the tunnel to reject all MCP traffic until a new one is added.

| Method | HTTP | Description |
|--------|------|-------------|
| `create` | `POST /v1/tunnels/{id}/certificates?beta=true` | Register a CA cert (PEM) |
| `retrieve` | `GET /v1/tunnels/{id}/certificates/{cert_id}?beta=true` | Fetch a certificate |
| `list` | `GET /v1/tunnels/{id}/certificates?beta=true` | List certs (archived excluded by default) |
| `archive` | `POST /v1/tunnels/{id}/certificates/{cert_id}/archive?beta=true` | Archive a certificate |

### Python Example

```python
import anthropic

client = anthropic.Anthropic()
BETA = ["mcp-tunnels-2026-06-22"]

# 1. Create a tunnel
tunnel = client.beta.tunnels.create(
    display_name="my-local-mcp",
    betas=BETA,
)
print(tunnel.id)      # 'tnl_...'
print(tunnel.domain)  # 'abc123.tunnels.anthropic.com'

# 2. Register your CA certificate
with open("ca.pem") as f:
    ca_pem = f.read()

cert = client.beta.tunnels.certificates.create(
    tunnel.id,
    ca_certificate_pem=ca_pem,
    betas=BETA,
)
print(cert.fingerprint)  # sha-256 hex

# 3. Reveal the connector token (treat as a credential)
token_obj = client.beta.tunnels.reveal_token(tunnel.id, betas=BETA)
connector_token = token_obj.tunnel_token  # Pass to your connector process

# 4. Later: rotate the token if compromised
new_token = client.beta.tunnels.rotate_token(
    tunnel.id,
    reason="Scheduled rotation",
    betas=BETA,
)

# 5. Archive when done (irreversible)
client.beta.tunnels.archive(tunnel.id, betas=BETA)
```

### TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();
const BETA = ['mcp-tunnels-2026-06-22'];

// Create a tunnel
const tunnel = await client.beta.tunnels.create({ betas: BETA });
console.log(tunnel.id, tunnel.domain);

// Register CA certificate
const cert = await client.beta.tunnels.certificates.create(tunnel.id, {
  ca_certificate_pem: caPem,
  betas: BETA,
});

// Reveal connector token
const tokenObj = await client.beta.tunnels.revealToken(tunnel.id, { betas: BETA });
const connectorToken = tokenObj.tunnel_token;  // treat as a credential

// Rotate token
await client.beta.tunnels.rotateToken(tunnel.id, { reason: 'Scheduled rotation', betas: BETA });
```

### BetaTunnel Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | `str` | Tunnel ID, prefixed `tnl_` |
| `type` | `"tunnel"` | Always `"tunnel"` |
| `domain` | `str` | Anthropic-assigned hostname; globally unique, never reused |
| `display_name` | `str \| None` | Optional human-readable name (1-255 chars) |
| `created_at` | `datetime` | RFC 3339 creation timestamp |
| `archived_at` | `datetime \| None` | RFC 3339 archival timestamp, or `None` if active |

### BetaTunnelToken Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | `str` | Stable ID for the current token value; changes on rotation |
| `tunnel_token` | `str` | Connector credential — treat as a secret |
| `type` | `"tunnel_token"` | Always `"tunnel_token"` |

### BetaTunnelCertificate Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | `str` | Certificate ID, prefixed `tcrt_` |
| `type` | `"tunnel_certificate"` | Always `"tunnel_certificate"` |
| `tunnel_id` | `str` | ID of the parent tunnel |
| `fingerprint` | `str` | Lowercase hex SHA-256 of the cert's DER encoding |
| `created_at` | `datetime` | RFC 3339 timestamp |
| `expires_at` | `datetime \| None` | RFC 3339 expiry, or `None` |
| `archived_at` | `datetime \| None` | Set when certificate is archived |

### Gotchas

- A tunnel rejects all MCP traffic until **at least one CA certificate** is registered.
- `reveal_token` fetches live — Anthropic does not store the token value.
- `rotate_token` does NOT sever established connections, only prevents new ones with the old token.
- Archiving a tunnel is **irreversible**: hostname retired, all certs archived, token invalidated.
- Max **2 non-archived certificates** per tunnel; archiving the last active cert blocks traffic.
- `tunnel_token` from Python v0.117.0+ is wrapped in `SecretStr` to prevent accidental exposure in tracebacks.
- Available in Python v0.117.0+ and TypeScript v0.112.0+.

---

## Related

- [Tool Use](./tool-use.md)
- [Managed Agents](./managed-agents.md)
