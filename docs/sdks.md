# SDKs (Python & TypeScript)

> **Last updated:** 2026-07-13

## Python SDK

**Package:** `anthropic`  
**Requires:** Python 3.9+  
**License:** MIT

### Installation

```bash
pip install anthropic
```

### Basic Usage

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.content[0].text)
```

### Async Client

```python
import asyncio
from anthropic import AsyncAnthropic

client = AsyncAnthropic()

async def main():
    response = await client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello!"}],
    )
    print(response.content[0].text)

asyncio.run(main())
```

### Configuration

```python
client = anthropic.Anthropic(
    api_key="sk-ant-...",
    max_retries=3,
    timeout=60.0,
    base_url="https://api.anthropic.com",
    default_headers={"X-Custom": "value"},
)
```

### Pagination

```python
# Auto-paginate
for batch in client.messages.batches.list():
    print(batch.id)

# Manual
page = client.messages.batches.list(limit=10)
while True:
    for batch in page.data:
        print(batch.id)
    if not page.has_next_page():
        break
    page = page.get_next_page()
```

### Raw HTTP Access

```python
response = client.messages.with_raw_response.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.headers["x-request-id"])
message = response.parse()
```

### AWS Bedrock

```python
from anthropic import AnthropicBedrock

client = AnthropicBedrock(
    aws_region="us-east-1",
    # Uses AWS credentials from env vars or IAM role
)

response = client.messages.create(
    model="anthropic.claude-sonnet-4-6-v1:0",  # Bedrock model ID format
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
```

### Google Vertex AI

```python
from anthropic import AnthropicVertex

client = AnthropicVertex(
    region="us-east5",
    project_id="your-gcp-project",
)
```

---

## TypeScript SDK

**Package:** `@anthropic-ai/sdk`  
**Requires:** Node.js 18+  
**License:** MIT

### Installation

```bash
npm install @anthropic-ai/sdk
```

### Basic Usage

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'Hello!' }],
});
console.log(response.content[0].text);
```

### Configuration

```typescript
const client = new Anthropic({
  apiKey: 'sk-ant-...',
  maxRetries: 3,
  timeout: 60 * 1000,
  defaultHeaders: { 'X-Custom': 'value' },
});
```

### Streaming

```typescript
// High-level
const stream = client.messages
  .stream({ model: 'claude-sonnet-4-6', max_tokens: 1024, messages: [...] })
  .on('text', (text) => process.stdout.write(text));
await stream.done();

// Raw SSE
const rawStream = await client.messages.create({
  model: 'claude-sonnet-4-6', max_tokens: 1024, stream: true, messages: [...],
});
for await (const event of rawStream) {
  if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
    process.stdout.write(event.delta.text);
  }
}
```

### Zod Structured Outputs

```typescript
import { zodOutputFormat } from '@anthropic-ai/sdk/helpers/zod';
import { z } from 'zod';

const Schema = z.object({ items: z.array(z.string()), count: z.number() });

const message = await client.messages.parse({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [{ role: 'user', content: 'List 3 fruits.' }],
  output_config: { format: zodOutputFormat(Schema) },
});

console.log(message.parsed_output?.items);
```

### Tool Runner (TypeScript)

```typescript
import { betaZodTool } from '@anthropic-ai/sdk/helpers/beta/zod';
import { z } from 'zod';

const weatherTool = betaZodTool({
  name: 'get_weather',
  description: 'Get current weather',
  inputSchema: z.object({ location: z.string() }),
  run: ({ location }) => `72°F in ${location}`,
});

const finalMessage = await client.beta.messages.toolRunner({
  model: 'claude-sonnet-4-6',
  tools: [weatherTool],
  messages: [{ role: 'user', content: 'Weather in Tokyo?' }],
  max_tokens: 1024,
});
```

### Cancellation (TypeScript)

```typescript
const controller = new AbortController();
setTimeout(() => controller.abort(), 5000);

try {
  const response = await client.messages.create(
    { model: 'claude-sonnet-4-6', max_tokens: 1024, messages: [...] },
    { signal: controller.signal }
  );
} catch (err) {
  if (err.name === 'AbortError') console.log('Request cancelled');
}
```

## Structured Outputs (Python)

```python
import pydantic

class Response(pydantic.BaseModel):
    items: list[str]
    count: int

parsed = client.messages.parse(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "List 3 fruits."}],
    output_format=Response,
)
print(parsed.parsed_output.items)
```

## TypeScript Middleware (v0.101.0+)

```typescript
const client = new Anthropic({
  middleware: [
    async (request, next) => {
      console.log('Before request:', request.url);
      const response = await next(request);
      console.log('After request:', response.status);
      return response;
    },
  ],
});
```

Middleware runs before request signing. Request timeout applies to the inner fetch only, not the full middleware chain.

## Client-Side Fallbacks Middleware (v0.108.0 Python / v0.103.0 TypeScript)

For API providers that don't support server-side fallbacks, you can implement client-side fallback logic. The `claude-fable-5` and `claude-mythos-5` models support server-side fallbacks on refusal natively; this middleware pattern is for third-party providers.

```python
# Python: client-side fallback middleware
import anthropic

class FallbackMiddleware:
    def __init__(self, fallback_model: str):
        self.fallback_model = fallback_model

    def __call__(self, request, next_handler):
        response = next_handler(request)
        if hasattr(response, 'stop_reason') and response.stop_reason == 'refusal':
            # Retry with fallback model
            request_data = request.copy()
            request_data['model'] = self.fallback_model
            return next_handler(request_data)
        return response
```

```typescript
// TypeScript: client-side fallbacks middleware (v0.103.0+)
const client = new Anthropic({
  middleware: [
    async (request, next) => {
      const response = await next(request);
      // Handle refusal by falling back to alternate model
      return response;
    },
  ],
});
```

**Server-side fallbacks** (claude-fable-5, claude-mythos-5): Anthropic's infrastructure automatically switches models when content policy triggers — no client code required. Set `fallback` param if/when the API exposes it.

## SDK Version History

| SDK | Version | Date | Changes |
|-----|---------|------|---------|
| Python | v0.116.0 | 2026-07-02 | Add `agent-memory-2026-07-22` beta header for Memory Stores API |
| Python | v0.115.1 | 2026-07-01 | Remove nonfunctional types from SDK |
| Python | v0.115.0 | 2026-06-30 | Managed Agents: event delta streaming, agent overrides, reverse pagination, vault credential injection scoping, agent and deployment webhook events |
| Python | v0.114.0 | 2026-06-30 | Add `claude-sonnet-5`; agent-toolset: allow absolute paths resolving inside workdir |
| Python | v0.113.0 | 2026-06-29 | Support `web_search_20260318` and `web_fetch_20260318` tool types; fix async `count_tokens` missing `output_format`/`output_config`; accept `user_profile_id` in `count_tokens` |
| Python | v0.112.0 | 2026-06-24 | `system.message` streaming events; `user_profile_id` param → `anthropic-user-profile-id` header (requires `user-profiles` beta); new refusal category; memory tool parent-dir fix |
| Python | v0.111.0 | 2026-06-18 | Tag refusal-fallback middleware requests with `fallback-refusal-middleware` |
| Python | v0.110.0 | 2026-06-18 | Support `code_execution_20260120` programmatic tool calling; Bedrock stream event type fix; x-stainless-helper fixes |
| Python | v0.109.2 | 2026-06-15 | Retired models removed from API and SDKs |
| Python | v0.109.1 | 2026-06-09 | Add `frontier_llm` refusal category |
| Python | v0.109.0 | 2026-06-09 | Managed Agents deployments; env var credentials support |
| Python | v0.108.0 | 2026-06-09 | Add claude-mythos-5, claude-fable-5; server-side fallbacks on refusal; client-side fallbacks middleware |
| Python | v0.107.1 | 2026-06-07 | Foundry x-api-key header fix |
| Python | v0.107.0 | 2026-06-06 | Managed Agents type updates |
| Python | v0.106.0 | 2026-06-05 | Mark claude-opus-4-1 deprecated; Foundry client fixes |
| Python | v0.105.0 | 2026-05-28 | Add claude-opus-4-8, mid-conversation system blocks, output_tokens_details |
| TypeScript | v0.111.0 | 2026-07-10 | Dreams API (`client.beta.dreams.*`); `evaluated_permission` (allow/ask/deny) on session tool-use events; idle bounding by server `stop_reason` in `SessionToolRunner` |
| TypeScript | v0.110.0 | 2026-07-02 | Add `agent-memory-2026-07-22` beta header for Memory Stores API |
| TypeScript | v0.109.1 | 2026-07-01 | Remove nonfunctional types from SDK |
| TypeScript | v0.109.0 | 2026-06-30 | Managed Agents: event delta streaming, agent overrides, reverse pagination, vault credential injection scoping, agent and deployment webhook events |
| TypeScript | v0.108.0 | 2026-06-30 | Add `claude-sonnet-5`; agent-toolset: allow absolute paths resolving inside workdir |
| TypeScript | v0.107.0 | 2026-06-29 | Support `web_search_20260318` and `web_fetch_20260318` tool types; accept `user_profile_id` in `count_tokens`; restore `BatchCreateParams.Request.params` type; bound symlink canonicalization hops in agent-toolset |
| TypeScript | v0.106.0 | 2026-06-24 | `system.message` streaming events; `user_profile_id` param → `anthropic-user-profile-id` header (requires `user-profiles` beta); new refusal category; x-stainless-helper single source fix |
| TypeScript | v0.105.0 | 2026-06-18 | Support `code_execution_20260120`; lazy parsing of partial tool JSON input during streaming; removed deprecated model refs |
| TypeScript | v0.104.2 | 2026-06-15 | Retired model identifiers removed |
| TypeScript | v0.104.1 | 2026-06-09 | Add `frontier_llm` refusal category |
| TypeScript | v0.104.0 | 2026-06-09 | Managed Agents deployments; env var credentials |
| TypeScript | v0.103.0 | 2026-06-09 | Add claude-mythos-5, claude-fable-5; server-side & client-side fallbacks middleware |
| TypeScript | v0.102.0 | 2026-06-06 | Managed Agents type updates; middleware before request signing |
| TypeScript | v0.101.0 | 2026-06-05 | Middleware support; streaming stop_details fix; scientific notation JSON fix |
| TypeScript | v0.100.0 | 2026-05-28 | Add claude-opus-4-8, mid-conversation system blocks, output_tokens_details |

## Related

- [Authentication](./authentication.md)
- [Messages API](./messages-api.md)
- [Streaming](./streaming.md)
- [Rate Limits & Errors](./rate-limits-errors.md)
