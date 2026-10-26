# Authentication & API Keys

> **Last updated:** 2026-10-26

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

### SDK Major Versions from Oct 21, 2026: `Authorization: Bearer` and One Credential

Python SDK 2.0.0+ and TypeScript SDK 1.0.0+ (and the other new SDK majors and `ant` CLI 2.0.0) send the API key as `Authorization: Bearer <key>` instead of `X-Api-Key`. Update any proxy, gateway, middleware or log filter that reads `X-Api-Key`. Exceptions: `AnthropicFoundry` (Python) and `@anthropic-ai/foundry-sdk` (TypeScript) still send `x-api-key`.

A client now holds **one** credential (`api_key`, `auth_token`, `credentials`, `config` or `profile`). Passing two raises `ValueError` in Python and throws `TypeError` in TypeScript. With none passed, the first set of `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, the profile `ANTHROPIC_PROFILE` names, the workload identity federation variables, and the active profile on disk is used.

```python
client = anthropic.Anthropic(api_key=key, auth_token=token)
# Python SDK 1.x: sent both headers
# Python SDK 2.0.0+: raises ValueError
```

Source: [Migrating to API version 2026-10-21, SDK and CLI changes](https://platform.claude.com/docs/en/api/migration-2026-10-21#sdk-changes). Full details in [api-version-2026-10-21.md](./api-version-2026-10-21.md).

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
| `x-api-key` | Yes | Your API key (or send it as `Authorization: Bearer <key>`; the Claude API accepts either) |
| `anthropic-version` | Yes | `2026-10-21` (latest, released Oct 21, 2026) or `2023-06-01` (still supported). See [api-version-2026-10-21.md](./api-version-2026-10-21.md) |
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

Now called **Claude Models on Google Cloud (Google managed)** in Anthropic's docs.

```python
client = anthropic.AnthropicVertex(
    region="us-east5",
    project_id="your-gcp-project",
)
```

### Claude Models on Google Cloud (Anthropic managed) (Oct 22, 2026)

The Claude API operated by Anthropic and reached through your Google Cloud project. Google Cloud IAM controls access (`roles/claude.user` to call, `roles/claude.admin` to open the Claude Console) and Google Cloud Marketplace handles billing. There are **no API keys**: requests use a Google access token as `Authorization: Bearer`, and the workspace ID goes in the URL, not an `anthropic-workspace-id` header. Model IDs are the same as on the Claude API. The SDK client is in beta.

```bash
pip install -U "anthropic[google_cloud]"      # Python
npm install @anthropic-ai/google-cloud-sdk    # TypeScript

export ANTHROPIC_GOOGLE_CLOUD_PROJECT="my-project-id"
export ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID="wrkspc_01AbCdEf23GhIj"
```

```python
from anthropic import AnthropicGoogleCloud

client = AnthropicGoogleCloud()

message = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
)
print(message)
```

```typescript
import { AnthropicGoogleCloud } from "@anthropic-ai/google-cloud-sdk";

const client = new AnthropicGoogleCloud();

const message = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }]
});
console.log(message);
```

- Base URL: `https://claude.googleapis.com/v1/projects/{project}/locations/global/workspaces/{workspace}/invoke` (organization routes use `.../locations/global/claudeOrganization/invoke`). The SDK builds it from the project, location (`global` by default) and workspace ID.
- Credentials: a token provider you pass, a Google credentials object, or Application Default Credentials (in that order). The client ignores `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` and `ANTHROPIC_BASE_URL`.
- Setup creates a new Anthropic organization per Google Cloud project; existing organizations can't be converted. Enable `claude.googleapis.com` and `anthropic-managed-claude.cloudpartnerservices.goog` on each calling project.
- Supported: fast mode (Opus 5.5), tool use (web search, web fetch, code execution, memory, advisor, MCP connector), Agent Skills, Files API, prompt caching, thinking, server-side fallback, streaming, structured outputs, token counting, Message Batches, Models API, Claude Managed Agents. Not supported: API keys, MCP tunnels, memory stores on self-hosted environments, the pre-built self-hosted sandbox workers.
- Billed through Google Cloud Marketplace in Claude Consumption Units ($0.01 each). Same data retention as the Claude API; ZDR on request. Rate limits are managed by Anthropic. Model lifecycle dates follow Anthropic's deprecations page.

Source: [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/en/build-with-claude/claude-models-on-google-cloud/anthropic-managed).

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

## API Credits for Max and Team Plans (Oct 7, 2026)

Claude Max and Team plans include monthly Claude API credits. To claim them, link a Claude Console organization to the plan; credits then arrive in that organization each billing cycle (monthly for annual plans). No payment method is needed on Claude Platform. Source: [API credits for Max and Team plans](https://platform.claude.com/docs/en/about-claude/api-credits-for-subscribers).

| Plan                                          | Monthly credit                                   |
| --------------------------------------------- | ------------------------------------------------ |
| Max 5x                                        | $100 USD                                         |
| Max 20x                                       | $200 USD                                         |
| Team, Standard seat                           | $20 USD per seat (pooled, capped at $500/month)  |
| Team, Premium seat                            | $100 USD per seat (pooled, capped at $500/month) |
| Discounted Team plans (Nonprofit, Scientists) | Same as Team                                     |

- **Covers:** Claude API, Claude Managed Agents, the Claude Agent SDK and the playground. **Doesn't cover:** Claude Code, extra usage in the Claude apps.
- Credits expire at the end of each billing cycle (no rollover) and apply only to the Claude API in the Claude Console, not Claude Platform on AWS, Bedrock, Google Cloud or Foundry.
- Eligibility: active plan in good standing; new subscribers after 7 days. Free, Pro and Enterprise plans aren't eligible. Claiming needs the subscriber (Max) or a Primary Owner / Owner (Team) plus an Owner, Admin or Billing role in the Console organization.

## Related

- [Rate Limits & Errors](./rate-limits-errors.md)
- [SDKs](./sdks.md)
- [Quick Reference](./QUICK-REFERENCE.md)
