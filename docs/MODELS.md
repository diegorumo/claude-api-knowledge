# Claude Models Reference

> **Last updated:** 2026-06-08  
> **Source:** GitHub SDK repos, SDK changelogs (Python v0.107.1, TypeScript v0.102.0)

## Current Model IDs

| Model | ID | Context Window | Max Output | Best For |
|-------|-----|----------------|------------|----------|
| Claude Opus 4.8 | `claude-opus-4-8` | 200K tokens | 32K tokens | Highest capability tier |
| Claude Opus 4.7 | `claude-opus-4-7` | 200K tokens | 32K tokens | High capability |
| Claude Mythos (preview) | `claude-mythos-preview` | TBD | TBD | New model family — preview |
| Claude Opus 4.6 | `claude-opus-4-6` | 200K tokens | 32K tokens | Recommended Opus — used in SDK examples |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K tokens | 64K tokens | Balanced speed/intelligence |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 200K tokens | 8K tokens | Fastest, most cost-efficient |

> **Note:** Always verify current pricing at https://www.anthropic.com/pricing — prices change and are not reproduced here to avoid stale data.
>
> **Note on claude-mythos-preview:** This model was identified in SDK type definitions (June 2026) but Anthropic has not published full capability details yet. Treat as experimental.

## Previous / Also Supported Models

| Model | ID | Status |
|-------|-----|--------|
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | Previous Opus; still works |
| Claude Opus 4.5 | `claude-opus-4-5` | Alias without date suffix |
| Claude Opus 4.1 | `claude-opus-4-1-20250805` | **Deprecated** (June 2026) |
| Claude Opus 4.1 | `claude-opus-4-1` | **Deprecated** (June 2026) |
| Claude Opus 4.0 | `claude-opus-4-20250514` | Older Opus 4 release |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | Previous Sonnet; still works |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | Alias without date suffix |
| Claude Sonnet 4.0 | `claude-sonnet-4-20250514` | Older Sonnet 4 release |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Alias for latest Haiku 4.5 |
| Claude 3 Haiku | `claude-3-haiku-20240307` | Legacy Claude 3; still works |

> **Deprecation notice:** `claude-opus-4-1` was marked deprecated in Python SDK v0.106.0 (2026-06-05). Migrate to `claude-opus-4-6` or higher.

## Model Capabilities

| Capability | Opus 4.6–4.8 | Sonnet 4.6 | Haiku 4.5 |
|-----------|--------------|------------|-----------|
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
