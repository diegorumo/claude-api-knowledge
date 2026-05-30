# Claude Models Reference

> **Last updated:** 2026-05-30  
> **Source:** GitHub SDK repos, SDK changelog (v0.105.0 adds claude-opus-4-8 on 2026-05-28)

## Current Model IDs

| Model | ID | Context Window | Max Output | Best For |
|-------|-----|----------------|------------|----------|
| Claude Opus 4.8 | `claude-opus-4-8` | 200K tokens | 32K tokens | Most capable, complex reasoning |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K tokens | 64K tokens | Balanced speed/intelligence |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 200K tokens | 8K tokens | Fastest, most cost-efficient |

> **Note:** Always verify current pricing at https://www.anthropic.com/pricing — prices change and are not reproduced here to avoid stale data.

## Previous / Also Supported Models

| Model | ID | Status |
|-------|-----|--------|
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | Previous Sonnet; still works in API |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | Alias without date suffix |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Alias for latest Haiku 4.5 |

> **Note:** SDK examples as of May 2026 use `claude-sonnet-4-5-20250929` — update to `claude-sonnet-4-6` for latest.

## Model Capabilities

| Capability | Opus 4.8 | Sonnet 4.6 | Haiku 4.5 |
|-----------|----------|------------|-----------|
| Extended Thinking | ✅ | ✅ | ❌ |
| Tool Use | ✅ | ✅ | ✅ |
| Vision (images) | ✅ | ✅ | ✅ |
| PDF Input | ✅ | ✅ | ✅ |
| Prompt Caching | ✅ | ✅ | ✅ |
| Streaming | ✅ | ✅ | ✅ |
| Batch API | ✅ | ✅ | ✅ |

## Prompt Caching Minimum Tokens

| Model Family | Minimum Cacheable Tokens |
|-------------|---------------------------|
| Sonnet models | 1,024 tokens |
| Opus / Haiku models | 4,096 tokens |

## Listing Available Models

```python
import anthropic

client = anthropic.Anthropic()
models = client.models.list()
for model in models:
    print(model.id)
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();
const models = await client.models.list();
for (const model of models.data) {
  console.log(model.id);
}
```

**API endpoint:** `GET /v1/models`  
**Single model:** `GET /v1/models/{model_id}`

## Related

- [Authentication](./authentication.md)
- [Messages API](./messages-api.md)
- [Prompt Caching](./prompt-caching.md)
- [Extended Thinking](./extended-thinking.md)
