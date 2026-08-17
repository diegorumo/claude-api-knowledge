# Claude Models Reference

> **Last updated:** 2026-08-17  
> **Source:** platform.claude.com/docs/en/about-claude/models/overview (Python v0.122.0, TypeScript v0.117.0)

## Current Models (Recommended)

| Model | API ID | Alias | Context | Max Output | Price (input/output MTok) | Best For |
|-------|--------|-------|---------|------------|---------------------------|----------|
| Claude Fable 5 | `claude-fable-5` | `claude-fable-5` | 1M tokens | 128k tokens | $10 / $50 | Most capable; long-running agents (always-on adaptive thinking) |
| Claude Opus 5 | `claude-opus-5` | `claude-opus-5` | 1M tokens | 128k tokens | $5 / $25 | Complex agentic coding and enterprise work |
| Claude Sonnet 5 | `claude-sonnet-5` | `claude-sonnet-5` | 1M tokens | 128k tokens | $2 / $10 | Best balance of speed and intelligence |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | `claude-haiku-4-5` | 200k tokens | 64k tokens | $1 / $5 | Fastest; high-volume, latency-sensitive apps |

> **Tokenizer note (Fable 5, Sonnet 5, Opus 4.7+):** Claude Fable 5 and Claude Sonnet 5 use the tokenizer introduced with Claude Opus 4.7. Compared to models before Opus 4.7, the same text produces **roughly 30% more tokens**. Measure your prompts with the [token counting API](./token-counting.md) when migrating.
>
> **Pricing note:** Sonnet 5 pricing is locked at $2/$10 per MTok (the scheduled Sept 1, 2026 increase to $3/$15 was cancelled on Aug 10, 2026).
>
> **Claude Mythos 5** (`claude-mythos-5`) shares Fable 5's specs and pricing but is invitation-only (Project Glasswing). Not generally available.

## Legacy / Also-Available Models

| Model | API ID | Context | Max Output | Price (input/output MTok) |
|-------|--------|---------|------------|---------------------------|
| Claude Opus 4.8 | `claude-opus-4-8` | 1M tokens | 128k tokens | $5 / $25 |
| Claude Opus 4.7 | `claude-opus-4-7` | 1M tokens | 128k tokens | $5 / $25 |
| Claude Opus 4.6 | `claude-opus-4-6` | 1M tokens | 128k tokens | $5 / $25 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M tokens | 128k tokens | $3 / $15 |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | 200k tokens | 64k tokens | $3 / $15 |
| Claude Opus 4.5 | `claude-opus-4-5-20251101` | 200k tokens | 64k tokens | $5 / $25 |

> **Note:** Claude Sonnet 4 (`claude-sonnet-4-20250514`) and Claude Opus 4 (`claude-opus-4-20250514`) were retired June 15, 2026.
> **Note:** Claude Opus 4.1 (`claude-opus-4-1-20250805`) was retired August 5, 2026. Migrate to `claude-opus-5`.
> **Extended thinking deprecated** on `claude-opus-4-6` and `claude-sonnet-4-6` (still accepted but use `effort` parameter on newer models instead).

## Retired Models (Return Error)

| Model | Retired |
|-------|---------|
| `claude-opus-4-1-20250805` / `claude-opus-4-1` | Aug 5, 2026 |
| `claude-sonnet-4-20250514` | Jun 15, 2026 |
| `claude-opus-4-20250514` | Jun 15, 2026 |

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
>
> **Retired models cleanup (June 2026):** Python SDK v0.109.2 and TypeScript SDK v0.104.2 (2026-06-15) removed retired model identifiers from the SDK type definitions. Any models not listed in the tables above should be considered unsupported.

## Model Capabilities

| Capability | Fable 5 / Mythos 5 | Opus 5 | Sonnet 5 | Haiku 4.5 | Opus 4.8 | Opus 4.6–4.7 | Sonnet 4.6 |
|-----------|---------------------|--------|----------|-----------|----------|--------------|------------|
| Adaptive thinking (always-on) | ✅ (always on) | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Extended thinking (explicit) | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ (deprecated) | ✅ (deprecated) |
| Tool Use | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Vision (images) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| PDF Input | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Prompt Caching | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Streaming | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Batch API | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Server-Side Fallbacks | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Computer Use | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mid-conv tool changes | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |

**Knowledge cutoffs:**
- Fable 5 / Mythos 5: reliable Jan 2026, training Jan 2026
- Opus 5: reliable May 2026, training May 2026  
- Sonnet 5: reliable Jan 2026, training Jan 2026
- Haiku 4.5: reliable Feb 2025, training Jul 2025
- Opus 4.8 / 4.7: reliable Jan 2026, training Jan 2026
- Opus 4.6: reliable May 2025, training Aug 2025
- Sonnet 4.6: reliable Aug 2025, training Jan 2026

**Effort defaults:**
- Opus 4.8: defaults to `high` on all surfaces
- Opus 5 / Sonnet 5: defaults to `high` on Claude API and Claude Code
- On Opus 5: `thinking: {type: "disabled"}` only allowed at effort `high` or below
- Fable 5: adaptive thinking is always on; `thinking: {type: "disabled"}` returns 400

## Prompt Caching Minimum Tokens

| Model | Minimum Cacheable Tokens |
|-------|--------------------------|
| Claude Opus 4.8+ (including Fable 5, Opus 5, Sonnet 5, Haiku 4.5) | 1,024 tokens |
| Claude Opus 4.7 and earlier | 4,096 tokens |

> **Batch API extended output:** Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5, and Sonnet 4.6 support up to **300k output tokens** on the Batch API using the `output-300k-2026-03-24` beta header.

## Platform Availability

| Platform | Notes |
|----------|-------|
| Claude API | All current models |
| Amazon Bedrock | Fable 5, Opus 5, Sonnet 5 (`anthropic.claude-*-53`); Haiku 4.5 (`anthropic.claude-haiku-4-5-20251001-v1:0`) |
| Google Cloud Vertex | Use same model IDs; Haiku 4.5 uses `claude-haiku-4-5@20251001` |
| Claude Platform on AWS | Same IDs as Claude API (not Bedrock-style); follows Anthropic deprecation schedule |
| Microsoft Foundry | Current models available |

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
