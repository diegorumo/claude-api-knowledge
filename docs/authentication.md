# Authentication & API Keys

> **Last updated:** 2026-08-31

## Overview

The Claude API uses API keys for authentication. All requests must include your API key.

## API Key Types (Updated Aug 27, 2026)

Three key types are now available in the Claude Console:

| Type | Tied to | Scope |
|------|---------|-------|
| **Workspace API key** | Workspace (legacy) | One workspace; anonymous |
| **Personal key** | Your account | Same permissions as your account; stops working if you leave the org |
| **Service account key** | A [service account](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation#service-accounts) | Same permissions as the service account; stops working if service account is removed |

Personal keys and service account keys can be scoped to a specific workspace or granted admin access across workspaces. Organization admins can track usage per account using personal/service-account keys. Workspace API keys remain supported as a legacy option.

## Getting an API Key

1. Sign in at https://console.anthropic.com
2. Navigate to **API Keys** in settings
3. Choose the key type (workspace key, personal key, or service account key)
4. Optionally set an expiration (preset, custom duration, or Never)
5. Create the key and store it securely — it won't be shown again

## Setting Your API Key

### Environment Variable (Recommended)

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

Both SDKs automatically read `ANTHROPIC_API_KEY` from the environment if no key is passed explicitly.

### Python SDK

```python
import anthropic

# Automatic (reads ANTHROPIC_API_KEY env var)
client = anthropic.Anthropic()

# Explicit
client = anthropic.Anthropic(api_key="sk-ant-...")
```

### TypeScript SDK

```typescript
import Anthropic from '@anthropic-ai/sdk';

// Automatic (reads ANTHROPIC_API_KEY env var)
const client = new Anthropic();

// Explicit
const client = new Anthropic({ apiKey: 'sk-ant-...' });
```

### Raw HTTP

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":1024,"messages":[{"role":"user","content":"Hello"}]}'
```

## HTTP Headers

### Request headers

| Header | Required | Value |
|--------|----------|-------|
| `x-api-key` | Yes | Your API key |
| `anthropic-version` | Yes | `2023-06-01` (current stable) |
| `content-type` | Yes | `application/json` |
| `anthropic-beta` | No | Beta features (e.g. `mcp-client-2025-04-04`) |

### Response headers

| Header | Description |
|--------|-------------|
| `request-id` | Unique ID for the request; include in bug reports |
| `anthropic-organization-id` | Your organization's UUID |
| `anthropic-workspace-id` | `wrkspc_`-prefixed ID of the workspace the API key resolved to (added Aug 11, 2026). Absent on Admin API requests and failed-auth responses. |

Use `anthropic-workspace-id` to confirm which workspace's rate limits and usage a request counted toward:

```python
response = client.messages.with_raw_response.create(
    model="claude-opus-5", max_tokens=100,
    messages=[{"role": "user", "content": "Hi"}],
)
workspace_id = response.headers.get("anthropic-workspace-id")
```

## Base URL

```
https://api.anthropic.com
```

## Third-Party Platform Auth

### AWS Bedrock

```python
import anthropic

client = anthropic.AnthropicBedrock(
    aws_region="us-east-1",
    # Uses AWS credentials from environment / IAM role
)
```

### Google Vertex AI

```python
client = anthropic.AnthropicVertex(
    region="us-east5",
    project_id="your-gcp-project",
)
```

### Azure

```python
client = anthropic.Anthropic(
    base_url="https://your-resource.openai.azure.com/",
    api_key="azure-api-key",
    default_headers={"api-key": "azure-api-key"},
)
```

## Security Notes

- Never commit API keys to source control
- Use environment variables or secrets managers (AWS Secrets Manager, HashiCorp Vault, etc.)
- Rotate keys if compromised — old key is immediately revoked
- Use separate keys per environment (dev, staging, prod)
- Keys are scoped to your Anthropic organization

## Related

- [Rate Limits & Errors](./rate-limits-errors.md)
- [SDKs](./sdks.md)
- [Quick Reference](./QUICK-REFERENCE.md)
