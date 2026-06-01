# Claude Models Reference

> **Last updated:** 2026-06-01  
> **Source:** GitHub SDK repos, anthropic-sdk-python v0.105.2 model.py

## Current Model IDs

| Model | ID | Context Window | Max Output | Best For |
|-------|-----|----------------|------------|----------|
| Claude Opus 4.8 | `claude-opus-4-8` | 200K tokens | 32K tokens | Most capable, complex reasoning |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K tokens | 64K tokens | Balanced speed/intelligence — recommended default |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 200K tokens | 8K tokens | Fastest, most cost-efficient |

> **Note:** Always verify current pricing at https://www.anthropic.com/pricing — prices change and are not reproduced here to avoid stale data.

## Preview / Experimental Models

| Model | ID | Notes |
|-------|-----|-------|
| Claude Mythos | `claude-mythos-preview` | Preview/experimental — subject to change |

## Previous / Also Supported Models

| Model | ID | Status |
|-------|-----|--------|
| Claude Opus 4.7 | `claude-opus-4-7` | Previous Opus; still works in API |
| Claude Opus 4.6 | `claude-opus-4-6` | Older Opus; still works in API |
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | Older Opus with date suffix |
| Claude Opus 4.5 | `claude-opus-4-5` | Alias without date suffix |
| Claude Opus 4.1 | `claude-opus-4-1-20250805` | Older Opus with date suffix |
| Claude Opus 4.1 | `claude-opus-4-1` | Alias without date suffix |
| Claude Opus 4.0 | `claude-opus-4-20250514` | Original Claude Opus 4 with date suffix |
| Claude Opus 4.0 | `claude-opus-4-0` | Alias for original Opus 4 |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | Previous Sonnet |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | Alias without date suffix |
| Claude Sonnet 4.0 | `claude-sonnet-4-20250514` | Original Claude Sonnet 4 |
| Claude Sonnet 4.0 | `claude-sonnet-4-0` | Alias for original Sonnet 4 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Alias for latest Haiku 4.5 |
| Claude 3 Haiku | `claude-3-haiku-20240307` | Claude 3-generation Haiku |

> **Note:** claude-opus-4-0 and claude-sonnet-4-0 (the original Claude 4 release) were marked deprecated in SDK v0.95.0 (2026-04-14). They remain callable but migrate to newer versions.

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
