# SDKs (Python & TypeScript)

> **Last updated:** 2026-05-30

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

## SDK Version History

- Python: v0.105.2 (2026-05-29) — latest
- TypeScript: aws-sdk v0.3.1 (2026-05-29) — latest

## Related

- [Authentication](./authentication.md)
- [Messages API](./messages-api.md)
- [Streaming](./streaming.md)
- [Rate Limits & Errors](./rate-limits-errors.md)
