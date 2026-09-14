# Structured Outputs

> **Last updated:** 2026-09-14  
> **Status:** GA (no beta header required)

## Overview

Structured outputs constrain Claude's responses to follow a specific JSON schema, guaranteeing valid, parseable output. Two complementary capabilities:

1. **JSON outputs** (`output_config.format`) — constrain Claude's entire response to a schema
2. **Strict tool use** (`strict: true` on a tool) — guarantee schema validation on tool inputs (see [Tool Use](./tool-use.md))

## Supported Models

`claude-fable-5-1`, `claude-mythos-5-1`, `claude-fable-5`, `claude-mythos-5`, `claude-mythos-preview`, `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7`, `claude-opus-4-6`, `claude-sonnet-5`, `claude-sonnet-4-6`, `claude-sonnet-4-5-20250929`, `claude-opus-4-5-20251101`, `claude-haiku-4-5-20251001`

## Key Benefits

- **Always valid JSON** — no parsing errors or missing fields
- **Type safe** — guaranteed field types and required fields  
- **No retries** — schema violations are structurally impossible

## Basic Usage

### Python (Pydantic)

```python
from pydantic import BaseModel
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan_interest: str
    demo_requested: bool

client = anthropic.Anthropic()

response = client.messages.parse(
    model="claude-opus-5",
    max_tokens=1024,
    output_format=ContactInfo,
    messages=[{
        "role": "user",
        "content": "Extract info from: John Smith (john@example.com) wants Enterprise plan and a demo."
    }],
)

print(response.parsed_output)
# ContactInfo(name='John Smith', email='john@example.com', plan_interest='Enterprise', demo_requested=True)
```

### Raw JSON Schema (cURL)

```bash
curl https://api.anthropic.com/v1/messages \
  -H "content-type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-5",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Extract: John Smith (john@example.com), Enterprise plan, wants demo"}],
    "output_config": {
      "format": {
        "type": "json_schema",
        "schema": {
          "type": "object",
          "properties": {
            "name":           {"type": "string"},
            "email":          {"type": "string"},
            "plan_interest":  {"type": "string"},
            "demo_requested": {"type": "boolean"}
          },
          "required": ["name", "email", "plan_interest", "demo_requested"],
          "additionalProperties": false
        }
      }
    }
  }'
```

### TypeScript (Zod)

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { z } from 'zod';
import { zodOutputFormat } from '@anthropic-ai/sdk/helpers/zod';

const ContactInfo = z.object({
  name: z.string(),
  email: z.string(),
  plan_interest: z.string(),
  demo_requested: z.boolean(),
});

const client = new Anthropic();

const response = await client.messages.parse({
  model: 'claude-opus-5',
  max_tokens: 1024,
  output_format: zodOutputFormat(ContactInfo, 'contact_info'),
  messages: [{
    role: 'user',
    content: 'Extract: John Smith (john@example.com), Enterprise plan, wants demo',
  }],
});

console.log(response.parsed_output);
```

## API Parameter

The `output_config.format` parameter goes in the request body:

```json
{
  "output_config": {
    "format": {
      "type": "json_schema",
      "schema": { ... }
    }
  }
}
```

The response's text content block contains valid JSON matching your schema.

## Common Use Cases

### Data Extraction

```python
class Invoice(BaseModel):
    invoice_number: str
    date: str
    total_amount: float
    line_items: list[dict]
    customer_name: str

response = client.messages.parse(
    model="claude-opus-5",
    max_tokens=4096,
    output_format=Invoice,
    messages=[{"role": "user", "content": f"Extract invoice data:\n{invoice_text}"}],
)
print(response.parsed_output.total_amount)
```

### Classification

```python
class Classification(BaseModel):
    category: str
    confidence: float
    tags: list[str]
    sentiment: str

response = client.messages.parse(
    model="claude-opus-5",
    max_tokens=1024,
    output_format=Classification,
    messages=[{"role": "user", "content": f"Classify: {feedback_text}"}],
)
```

### Combined with Strict Tool Use

```python
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Help me plan a trip to Paris departing May 15"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "summary":    {"type": "string"},
                    "next_steps": {"type": "array", "items": {"type": "string"}},
                },
                "required": ["summary", "next_steps"],
                "additionalProperties": False,
            },
        }
    },
    tools=[{
        "name": "search_flights",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date":        {"type": "string", "format": "date"},
            },
            "required": ["destination", "date"],
            "additionalProperties": False,
        },
    }],
)
```

## SDK-Specific Methods

| Language | Method | Notes |
|----------|---------|-------|
| Python | `client.messages.parse(output_format=MyModel)` | Pydantic models; returns `parsed_output` |
| TypeScript | `client.messages.parse()` + `zodOutputFormat()` | Zod schemas with type inference |
| TypeScript | `jsonSchemaOutputFormat()` | Raw JSON schema with optional type inference |
| Java | `outputConfig(Class<T>)` | Automatic schema from Java class; returns `StructuredMessage<T>` |
| Ruby | `output_config: {format: MyModel}` | Extend `Anthropic::BaseModel` |
| PHP | Implement `StructuredOutputModel` | Uses PHP 8 property types |
| C# | `Create<T>()` | Plain C# classes; parse with `JsonSerializer.Deserialize()` |
| Go | Struct tags + `invopop/jsonschema` | Parse with `json.Unmarshal()` |

## JSON Schema Support

### Supported

- All basic types: `object`, `array`, `string`, `integer`, `number`, `boolean`, `null`
- `enum` (strings, numbers, bools, or nulls only)
- `const`, `anyOf`, `allOf` (with limitations)
- `$ref`, `$def`, `definitions`
- String formats: `date-time`, `time`, `date`, `duration`, `email`, `hostname`, `uri`, `ipv4`, `ipv6`, `uuid`
- Array `minItems` (values 0 and 1 only)

### Not Supported

- Recursive schemas
- Complex types within enums
- External `$ref`
- Numerical constraints (`minimum`, `maximum`, `multipleOf`)
- String constraints (`minLength`, `maxLength`)
- Array constraints beyond `minItems`
- `additionalProperties` set to anything other than `false`

## Performance Gotchas

| Issue | Detail |
|-------|--------|
| **First-request latency** | Grammar compilation adds latency on the first call with a new schema |
| **Auto-caching** | Compiled grammars are cached for 24 hours |
| **Cache invalidation** | Any change to `output_config.format` or the tool set invalidates the grammar cache |
| **Token cost** | Structured outputs add a system prompt, slightly increasing input token count |
| **Prompt cache** | Changing `output_config.format` invalidates the prompt cache for that conversation |

## Migration from Beta

- Parameter moved from `output_format` (beta) to `output_config.format` (GA)
- No beta header required in GA
- Python SDK v1.0+: use `output_config` on `client.messages.create()`, not `client.beta.messages.create()`

## Related

- [Tool Use](./tool-use.md) — `strict: true` for strict tool input schemas
- [Messages API](./messages-api.md) — full request/response reference
- [Prompt Caching](./prompt-caching.md) — caching interactions with structured outputs
