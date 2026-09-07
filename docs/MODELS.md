# Claude Models Reference

> **Last updated:** 2026-09-07  
> **Source:** platform.claude.com/docs/en/about-claude/models/overview (Python v1.4.0, TypeScript v0.124.0)

## Current Models (Recommended)

| Model | API ID | Alias | Context | Max Output | Price (input/output MTok) | Best For |
|-------|--------|-------|---------|------------|---------------------------|----------|
| Claude Fable 5.1 | `claude-fable-5-1` | `claude-fable-5-1` | 1M tokens | 128k tokens | $10 / $50 | Most capable; demanding reasoning and long-horizon agentic work (always-on adaptive thinking) |
| Claude Opus 5 | `claude-opus-5` | `claude-opus-5` | 1M tokens | 128k tokens | $5 / $25 | Complex agentic coding and enterprise work |
| Claude Sonnet 5 | `claude-sonnet-5` | `claude-sonnet-5` | 1M tokens | 128k tokens | $2 / $10 | Best balance of speed and intelligence |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | `claude-haiku-4-5` | 200k tokens | 64k tokens | $1 / $5 | Fastest; high-volume, latency-sensitive apps |

> **Tokenizer note (Fable 5.1, Fable 5, Sonnet 5, Opus 4.7+):** These models use the tokenizer introduced with Claude Opus 4.7. Compared to models before Opus 4.7, the same text produces **roughly 30% more tokens**. Measure your prompts with the [token counting API](./token-counting.md) when migrating.
>
> **Pricing note:** Sonnet 5 pricing is locked at $2/$10 per MTok (the scheduled Sept 1, 2026 increase to $3/$15 was cancelled on Aug 10, 2026).
>
> **Claude Mythos 5.1** (`claude-mythos-5-1`) shares Fable 5.1's specs and pricing but is invitation-only (Project Glasswing). Not generally available.
>
> **Claude Fable 5** (`claude-fable-5`) is now legacy (still available) — see Legacy table below.

## ⚠️ Fable 5.1 / Mythos 5.1 API Restrictions

These models have specific constraints not present on earlier models:

**`tool_choice` restrictions:**
- `tool_choice: {type: "any"}` and `tool_choice: {type: "tool"}` are **not supported** (return 400)
- `tool_choice: {type: "auto"}` and `tool_choice: {type: "none"}` remain available
- For schema-constrained outputs, use [structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) instead

**Thinking block binding:**
- Thinking blocks are preserved only for the model that produced them or newer models
- The API silently drops thinking blocks replayed to an older model
- **New accounts (created on/after Aug 31, 2026):** Replaying thinking blocks after changing `system`, `tools`, or earlier messages returns 400
- Use beta header `thinking-binding-controls-2026-08-01` to get reports of dropped blocks in `input_transformations` and to control behavior via `thinking.block_binding.prefix_mismatch_behavior`

**Prompt cache read pricing:**
- Cache reads cost **2.5% of base input price** ($0.25/MTok) — vs. 10% on other models

**Data retention:**
- Fable 5.1 and Mythos 5.1 require **30-day data retention**
- Not available under zero data retention unless expressly authorized by Anthropic

**Content watermarking:**
- Text output carries Anthropic's text watermark
- Images, video, audio produced via code execution carry C2PA Content Credentials when retrieved via Files API

## Legacy / Also-Available Models

| Model | API ID | Context | Max Output | Price (input/output MTok) |
|-------|--------|---------|------------|---------------------------|
| Claude Fable 5 | `claude-fable-5` | 1M tokens | 128k tokens | $10 / $50 |
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

| Capability | Fable 5.1 / Mythos 5.1 | Fable 5 / Mythos 5 | Opus 5 | Sonnet 5 | Haiku 4.5 | Opus 4.8 | Opus 4.6–4.7 | Sonnet 4.6 |
|-----------|------------------------|---------------------|--------|----------|-----------|----------|--------------|------------|
| Adaptive thinking (always-on) | ✅ (always on) | ✅ (always on) | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| Extended thinking (explicit) | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ (deprecated) | ✅ (deprecated) |
| Tool Use | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `tool_choice: any/tool` | ❌ (400 error) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Vision (images) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| PDF Input | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Prompt Caching | ✅ (2.5% reads) | ✅ (10% reads) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Streaming | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Batch API | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Server-Side Fallbacks | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Computer Use | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Mid-conv tool changes | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Content Watermarking | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

**Knowledge cutoffs:**
- Fable 5.1 / Mythos 5.1: reliable Jun 2026, training Jun 2026
- Fable 5 / Mythos 5: reliable Jan 2026, training Jan 2026
- Opus 5: reliable May 2026, training May 2026  
- Sonnet 5: reliable Jan 2026, training Jan 2026
- Haiku 4.5: reliable Feb 2025, training Jul 2025
- Opus 4.8 / 4.7: reliable Jan 2026, training Jan 2026
- Opus 4.6: reliable May 2025, training Aug 2025
- Sonnet 4.6: reliable Aug 2025, training Jan 2026

**Effort defaults:**
- Opus 4.8: defaults to `high` on all surfaces
- Opus 5 / Sonnet 5 / Fable 5.1: defaults to `high` on Claude API and Claude Code
- Fable 5 / Fable 5.1: adaptive thinking is always on; `thinking: {type: "disabled"}` returns 400

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
