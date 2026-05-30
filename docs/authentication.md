# Authentication & API Keys

> **Last updated:** 2026-05-30

## Overview

The Claude API uses API keys for authentication. All requests must include your API key.

## Getting an API Key

1. Sign in at https://console.anthropic.com
2. Navigate to **API Keys** in settings
3. Create a new key and store it securely — it won't be shown again

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

| Header | Required | Value |
|--------|----------|-------|
| `x-api-key` | Yes | Your API key |
| `anthropic-version` | Yes | `2023-06-01` (current stable) |
| `content-type` | Yes | `application/json` |
| `anthropic-beta` | No | Beta features (e.g. `mcp-client-2025-04-04`) |

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
