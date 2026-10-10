# Claude Models Reference

> **Last updated:** 2026-10-10  
> **Source:** platform.claude.com/docs/en/about-claude/models/overview, platform.claude.com/docs/en/about-claude/model-deprecations, platform.claude.com/docs/en/about-claude/pricing, platform.claude.com/docs/en/build-with-claude/prompt-caching, platform.claude.com/docs/en/api/models/list, platform.claude.com/docs/en/release-notes/overview (Sep 22 – Oct 7, 2026), platform.claude.com/docs/en/models/haiku-5-5/{overview,whats-new-haiku-5-5,migration-guide}, platform.claude.com/docs/en/models/sonnet-5-5/{overview,whats-new-sonnet-5-5,migration-guide}, platform.claude.com/docs/en/models/sonnet-4-5/overview

## Current Models (Recommended)

| Model            | API ID                      | Alias              | Context     | Max Output  | Price (input/output MTok) | Best For                                                                                                                                                 |
| ---------------- | --------------------------- | ------------------ | ----------- | ----------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Fable 5.1 | `claude-fable-5-1`          | `claude-fable-5-1` | 1M tokens   | 128k tokens | $10 / $50                 | Most capable; demanding reasoning and long-horizon agentic work (always-on adaptive thinking)                                                            |
| Claude Opus 5.5  | `claude-opus-5-5`           | `claude-opus-5-5`  | 1M tokens   | 128k tokens | $4 / $20                  | **Recommended starting point for most workloads**; long-running agentic coding and knowledge work (always-on adaptive thinking, default effort `medium`) |
| Claude Sonnet 5.5 | `claude-sonnet-5-5`        | `claude-sonnet-5-5` | 1M tokens  | 128k tokens | $2 / $10                  | Best balance of speed and intelligence (adaptive thinking, default effort `high`); launched Sep 28, 2026                                                 |
| Claude Haiku 5.5 | `claude-haiku-5-5`          | `claude-haiku-5-5` | 1M tokens   | 128k tokens | From $0.10 / $0.50 (see note) | Fastest; high-volume, latency-sensitive work such as classification, extraction and routing (adaptive thinking, default effort `medium`); launched Oct 7, 2026 |

> **Tokenizer note (Fable 5.1, Fable 5, Sonnet 5.5, Sonnet 5, Haiku 5.5, Opus 4.7+):** These models use the tokenizer introduced with Claude Opus 4.7. Compared to models before Opus 4.7, the same text produces **roughly 30% more tokens**. Measure your prompts with the [token counting API](./token-counting.md) when migrating.
>
> **Haiku 5.5 pricing (prompt-length tiers):** $0.10 input / $0.50 output per MTok for prompts up to 100,000 tokens; $0.50 / $2.50 for prompts over 100,000 tokens. A request's prompt length counts all input tokens, including cache reads and writes. Batch API: $0.05 / $0.25 and $0.25 / $1.25. Haiku 5.5 is the only current model that does not get the full 1M context at standard pricing.
>
> **Pricing note:** Sonnet 5 pricing is locked at $2/$10 per MTok (the scheduled Sept 1, 2026 increase to $3/$15 was cancelled on Aug 10, 2026).
>
> **Claude Mythos 5.1** (`claude-mythos-5-1`) shares Fable 5.1's specs and pricing but is invitation-only (Project Glasswing). Not generally available.
>
> **Claude Fable 5** (`claude-fable-5`), **Claude Opus 5** (`claude-opus-5`), **Claude Sonnet 5** (`claude-sonnet-5`, since Sep 28, 2026) and **Claude Haiku 4.5** (`claude-haiku-4-5-20251001`, since Oct 7, 2026) are now legacy (still available) — see Legacy table below.
>
> **Retirement commitments (Claude API, Claude Platform on AWS, Foundry):** Fable 5.1 not sooner than Sep 1, 2027; Opus 5.5 not sooner than Sep 22, 2027; Sonnet 5.5 not sooner than Sep 28, 2027; Haiku 5.5 not sooner than Oct 7, 2027. Bedrock and Google Cloud set their own dates.
>
> **Choosing:** Anthropic's models overview recommends starting with Opus 5.5 for most workloads and moving to Fable 5.1 for demanding reasoning / long-horizon agentic work, or when Opus 5.5 at higher effort still falls short.

## ⚠️ Haiku 5.5 API Changes (launched Oct 7, 2026)

Claude Haiku 5.5 (`claude-haiku-5-5`) succeeds Haiku 4.5. 1M context (Haiku 4.5: 200k), 128k max output (Haiku 4.5: 64k), adaptive thinking with the effort parameter (default `medium`), reliable knowledge cutoff Jun 2026. Available on Claude API, Amazon Bedrock (`anthropic.claude-haiku-5-5`), Google Cloud, Microsoft Foundry and Claude Platform on AWS (all `claude-haiku-5-5`). `claude-haiku-5-5` is a fixed ID with no date suffix and no separate alias. Retirement not sooner than Oct 7, 2027.

Code written for Haiku 4.5 can break (per the Oct 7 release note and the migration guide):

- **Manual extended thinking rejected.** `thinking: {type: "enabled", budget_tokens: N}` returns 400. Use `{type: "adaptive"}` (or omit `thinking`) and set `output_config.effort`. `thinking: {type: "disabled"}` still works at `high` effort or below.
- **Sampling parameters.** Omit `temperature`, `top_p` and `top_k`. A `temperature` other than `1`, a `top_p` other than `0.99` (including `1`), any `top_k`, or `temperature` and `top_p` together return 400.
- **Assistant prefill rejected** (400), even with thinking off. End `messages` with a user turn.
- **`computer_20250124` rejected** on every platform. Use `computer_toolset_20260801` on the Claude API and Google Cloud, or `computer_20251124` (beta `computer-use-2025-11-24`) on Amazon Bedrock.
- **Thinking blocks are bound to the conversation.** Sending a thinking block back after a change to `system`, `tools` or earlier turns returns 400 (accounts created before Aug 31, 2026 only get the error when they set `thinking.block_binding.prefix_mismatch_behavior`). Keep conversations append-only. Blocks also only work in the account that produced them (or a linked account).
- **Structured outputs are not available on Amazon Bedrock** for Haiku 5.5.

Behavior changes:

- Adaptive thinking is on by default, so a response can begin with `thinking` blocks. Select content blocks by `type`, not position. Thinking text is omitted by default; set `thinking.display: "summarized"` to get it.
- Thinking tokens count toward `max_tokens`; a small limit can stop after a `thinking` block with no text.
- Thinking blocks from all earlier assistant turns stay in context and count as input (Haiku 4.5 kept only the latest turn's).
- Same text is about 30% more tokens (Opus 4.7+ tokenizer). Large images also cost more: high-resolution image tier (downscale above 2,576 px long edge or 4,784 visual tokens).
- Forced `tool_choice` (`any` / named tool) is accepted, but the response starts with the tool call and has no `thinking` block.
- Safety classifiers can decline a request (`stop_reason: "refusal"`); no server-side fallback.
- Minimum cacheable prompt **512 tokens** (Haiku 4.5: 4,096). Context awareness tags aren't injected; use task budgets (beta) instead.
- Supports the browser use tool (`browser_toolset_20260801`) on the Claude API and Google Cloud. Priority Tier is not supported.
- Reads thinking blocks from Sonnet 5, Opus 4.8, Haiku 4.5 and earlier; not from Opus 5, Opus 5.5, Sonnet 5.5 or any Fable / Mythos model.

```python
# Before (Haiku 4.5)
client.messages.create(
    model="claude-haiku-4-5",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    messages=[{"role": "user", "content": "..."}],
)

# After (Haiku 5.5): adaptive thinking + effort
client.messages.create(
    model="claude-haiku-5-5",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "medium"},
    messages=[{"role": "user", "content": "..."}],
)
```

```typescript
await client.messages.create({
  model: "claude-haiku-5-5",
  max_tokens: 16000,
  thinking: { type: "adaptive" },
  output_config: { effort: "medium" },
  messages: [{ role: "user", content: "..." }],
});
```

Details: [What's new in Claude Haiku 5.5](https://platform.claude.com/docs/en/models/haiku-5-5/whats-new-haiku-5-5), [migration guide](https://platform.claude.com/docs/en/models/haiku-5-5/migration-guide). See also [migrations.md](./migrations.md).

## ⚠️ Sonnet 5.5 API Changes (launched Sep 28, 2026)

Claude Sonnet 5.5 (`claude-sonnet-5-5`) succeeds Sonnet 5 at the same $2 / $10 per MTok. Cache reads are **$0.10/MTok (5% of base input) since Oct 7, 2026** (were $0.20, 10%); cache writes unchanged. 1M context, 128k max output, reliable knowledge cutoff Jun 2026. Available on Claude API, Amazon Bedrock (`anthropic.claude-sonnet-5-5`), Google Cloud, Microsoft Foundry and Claude Platform on AWS (all `claude-sonnet-5-5`). Retirement not sooner than Sep 28, 2027.

Code written for Sonnet 5 can break in five ways (per the Sep 28 release note):

- **No `thinking: {type: "disabled"}`.** To turn off up-front thinking, send `thinking: {type: "between_tools"}` instead, at `high` effort or below.
- **Forced tool use rejected.** `tool_choice` types `any` and `tool` return 400.
- **Thinking blocks are tied to the model and the conversation.** Blocks Sonnet 5.5 produces also only work in the account that produced them (or a linked account); from another account the API drops them and the request still succeeds.
- **`computer_20251124` not accepted** on the Claude API and Google Cloud.
- **Advisor tool** rejects Opus 4.8, Opus 4.7 and Sonnet 5 as advisors.

More detail from the what's-new and migration pages:

- **`between_tools`** is the lowest thinking setting. No beta header; works on every platform that offers Sonnet 5.5. Accepted at `low`, `medium` and `high` effort only; at `xhigh` or `max` it returns 400 (use adaptive thinking there). It takes no other field (`display`, `budget_tokens` or `block_binding` with it returns 400), and with it effort can't change mid-conversation (a differing per-message `output_config.effort` returns 400). Progress updates between tool calls still come back as `thinking` blocks with summary text; pass them back unchanged. Without tools, the response is text only. With server-side fallback, a `between_tools` request that falls back to Sonnet 5 runs there with `thinking: {type: "disabled"}`.
- **Manual budgets rejected.** `thinking: {type: "enabled", budget_tokens: N}` returns 400. Default `display` is `"omitted"`.
- **Sampling parameters rejected.** A non-default `temperature`, `top_p` or `top_k` returns 400.
- **Forced `tool_choice`** is also rejected by the token counting endpoint.
- **Thinking-block reading rules.** Sonnet 5.5 reads blocks from Sonnet 5, Opus 4.8, Haiku 4.5 and earlier, but not from Opus 5, Opus 5.5 or any Fable / Mythos model. No other model reads Sonnet 5.5 blocks. Unreadable blocks are dropped (request succeeds, not billed). The prefix check (400 on edited `system` / `tools` / earlier messages) is on by default for accounts created on or after Aug 31, 2026 (Claude API, Bedrock, Google Cloud). `block_binding` works only with `adaptive`; with `between_tools`, keep history append-only.
- **Computer use:** `computer_20251124` is still accepted on **Amazon Bedrock**; on the Claude API and Google Cloud use `computer_toolset_20260801`.
- **Advisor tool:** accepted advisors are Mythos 5.1, Fable 5.1, Mythos 5, Fable 5, Opus 5.5, Opus 5, or Sonnet 5.5 itself. Advice comes back encrypted as `advisor_redacted_result`.
- **Text between tool calls** longer than a sentence or two comes back as progress-update `thinking` blocks (empty at the default `display`). Set `display: "updates"` (beta, `thinking-display-updates-2026-08-18`) or `"summarized"`, or use `between_tools`.
- **Refusal categories:** `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`. Server-side fallback (`fallbacks: "default"`, beta, Claude API only) retries `cyber` and `frontier_llm` declines on Sonnet 5.
- **Other:** same tokenizer as Sonnet 5 (same token counts); minimum cacheable prompt **512 tokens** (Sonnet 5: 1,024); cache writes $2.50 (5m) / $4 (1h) per MTok; up to 300k output on the Batch API with `output-300k-2026-03-24`. Effort levels are recalibrated vs. Sonnet 5, so re-run your effort sweep.
- **New vs. Sonnet 5:** per-message effort (beta), mid-conversation system messages and mid-conversation tool changes (beta).

```python
# Before (Sonnet 5)
client.messages.create(
    model="claude-sonnet-5",
    max_tokens=16000,
    thinking={"type": "disabled"},
    messages=[{"role": "user", "content": "..."}],
)

# After (Sonnet 5.5): between_tools, at high effort or below
client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=16000,
    thinking={"type": "between_tools"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}],
)
```

```typescript
await client.messages.create({
  model: "claude-sonnet-5-5",
  max_tokens: 16000,
  thinking: { type: "between_tools" },
  output_config: { effort: "high" },
  messages: [{ role: "user", content: "..." }],
});
```

Details: [What's new in Claude Sonnet 5.5](https://platform.claude.com/docs/en/models/sonnet-5-5/whats-new-sonnet-5-5), [migration guide](https://platform.claude.com/docs/en/models/sonnet-5-5/migration-guide). SDK support: Python v1.9.0 / TypeScript v0.129.0.

## ⚠️ Opus 5.5 API Changes (launched Sep 22, 2026)

Claude Opus 5.5 succeeds Opus 5 at a lower price ($4 / $20 vs. $5 / $25 per MTok). Same 1M context, 128k max output and tokenizer as Opus 5. Available on Claude API, Amazon Bedrock (`anthropic.claude-opus-5-5`), Google Cloud, Microsoft Foundry and Claude Platform on AWS (all `claude-opus-5-5`). Retirement not sooner than Sep 22, 2027.

Breaking changes for code moving from Opus 5:

- **Thinking can't be disabled.** `thinking: {type: "disabled"}` and `thinking: {type: "enabled", budget_tokens: N}` both return 400 at every effort level. Omit `thinking` (or send `{type: "adaptive"}`) and control depth with `output_config.effort`.
- **Default effort is `medium`** (Opus 5 defaults to `high`). Set effort explicitly if you relied on the old default.
- **Forced tool use rejected.** `tool_choice: {type: "any"}` and `{type: "tool"}` return 400. Use `auto` + `strict: true` and steer from the prompt, or use structured outputs.
- **Thinking blocks are bound to the model and conversation** (same "preserved thinking" rules as Fable 5.1, below). A fallback from Opus 5.5 to Opus 5 runs without Opus 5.5's thinking blocks.
- **Computer use requires `computer_toolset_20260801`.** The older `computer_20251124` tool returns 400 on the Claude API and Google Cloud.

Other notes:

- Prompt cache reads cost **5% of base input** ($0.20/MTok).
- Text between tool calls comes back as progress-update `thinking` blocks (empty unless `thinking.display: "updates"`).
- Broader safety classifiers: `bio` and `reasoning_extraction` refusal categories join `cyber`.
- **Fast mode** (research preview, Claude API only): $8 / $40 per MTok.

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    output_config={"effort": "high"},  # default is "medium"
    messages=[{"role": "user", "content": "..."}],
)
```

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

| Model             | API ID                                                   | Context     | Max Output  | Price (input/output MTok) | Retirement (not sooner than) |
| ----------------- | -------------------------------------------------------- | ----------- | ----------- | ------------------------- | ---------------------------- |
| Claude Fable 5    | `claude-fable-5`                                         | 1M tokens   | 128k tokens | $10 / $50                 | Jun 9, 2027                  |
| Claude Opus 5     | `claude-opus-5`                                          | 1M tokens   | 128k tokens | $5 / $25                  | Jul 24, 2027                 |
| Claude Sonnet 5   | `claude-sonnet-5`                                        | 1M tokens   | 128k tokens | $2 / $10                  | Jun 30, 2027                 |
| Claude Opus 4.8   | `claude-opus-4-8`                                        | 1M tokens   | 128k tokens | $5 / $25                  | May 28, 2027                 |
| Claude Opus 4.7   | `claude-opus-4-7`                                        | 1M tokens   | 128k tokens | $5 / $25                  | Apr 16, 2027                 |
| Claude Opus 4.6   | `claude-opus-4-6`                                        | 1M tokens   | 128k tokens | $5 / $25                  | Feb 5, 2027                  |
| Claude Sonnet 4.6 | `claude-sonnet-4-6`                                      | 1M tokens   | 128k tokens | $3 / $15                  | Feb 17, 2027                 |
| Claude Haiku 4.5  | `claude-haiku-4-5-20251001` (alias `claude-haiku-4-5`)   | 200k tokens | 64k tokens  | $1 / $5                   | **Oct 15, 2026**             |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` (alias `claude-sonnet-4-5`) | 200k tokens | 64k tokens  | $3 / $15                  | **Deprecated; retires Nov 30, 2026** |
| Claude Opus 4.5   | `claude-opus-4-5-20251101` (alias `claude-opus-4-5`)     | 200k tokens | 64k tokens  | $5 / $25                  | **Nov 24, 2026**             |

> Retirement dates are from the model deprecations page and apply to Anthropic-operated platforms (Claude API, Claude Platform on AWS, Microsoft Foundry). All models above are listed there as "Active" except Claude Sonnet 4.5.
>
> **Claude Sonnet 4.5 deprecated (Sep 30, 2026):** `claude-sonnet-4-5-20250929` (alias `claude-sonnet-4-5`) still works but retires on the Claude API on **Nov 30, 2026** (a fixed date, not "not sooner than"). Recommended replacement: `claude-sonnet-5-5`. See the [Sonnet 5.5 migration guide, "Migrating from Claude Sonnet 4.5 or earlier"](https://platform.claude.com/docs/en/models/sonnet-5-5/migration-guide#migrating-from-sonnet-45): it is not a drop-in swap (prefill, `budget_tokens` thinking and non-default `temperature`/`top_p`/`top_k` return 400 on Sonnet 5.5; about 30% more tokens for the same text). It is no longer in the models overview's "Legacy models (still available)" list. Amazon Bedrock and Google Cloud set their own dates.
>
> **Claude Mythos Preview** (`claude-mythos-preview`, invitation-only) was deprecated Jun 9, 2026; retirement date to be announced.
>
> **Extended thinking deprecated** on `claude-opus-4-6` and `claude-sonnet-4-6` (still accepted but use `effort` parameter on newer models instead).

## Retired Models (Return Error)

| Model                                          | Deprecated   | Retired      | Official replacement        |
| ---------------------------------------------- | ------------ | ------------ | --------------------------- |
| `claude-opus-4-1-20250805` / `claude-opus-4-1` | Jun 5, 2026  | Aug 5, 2026  | `claude-opus-4-8`           |
| `claude-sonnet-4-20250514`                     | Apr 14, 2026 | Jun 15, 2026 | `claude-sonnet-4-6`         |
| `claude-opus-4-20250514`                       | Apr 14, 2026 | Jun 15, 2026 | `claude-opus-4-8`           |
| `claude-3-haiku-20240307`                      | Feb 19, 2026 | Apr 20, 2026 | `claude-haiku-4-5-20251001` |
| `claude-3-7-sonnet-20250219`                   | Oct 28, 2025 | Feb 19, 2026 | `claude-sonnet-4-6`         |
| `claude-3-5-haiku-20241022`                    | Dec 19, 2025 | Feb 19, 2026 | `claude-haiku-4-5-20251001` |

> Older models (Claude 3.5 Sonnet, Claude 3 Opus, Claude 3 Sonnet, Claude 2.x, Claude 1.x / Instant) were retired between Nov 2024 and Jan 2026. Replacements above are the ones named on the deprecations page; a current model from the top table is usually the better target.
>
> **Retired models cleanup (June 2026):** Python SDK v0.109.2 and TypeScript SDK v0.104.2 (2026-06-15) removed retired model identifiers from the SDK type definitions. Any models not listed in the tables above should be considered unsupported.

## Model Capabilities

| Capability                    | Fable 5.1 / Mythos 5.1 | Opus 5.5                             | Sonnet 5.5                                                 | Haiku 5.5 | Fable 5 / Mythos 5 | Opus 5 | Sonnet 5 | Haiku 4.5 | Opus 4.8 | Opus 4.6–4.7   | Sonnet 4.6     |
| ----------------------------- | ---------------------- | ------------------------------------ | ---------------------------------------------------------- | ---------- | ------------------ | ------ | -------- | --------- | -------- | -------------- | -------------- |
| Adaptive thinking (always-on) | ✅ (always on)          | ✅ (always on)                        | ✅ (default on; `between_tools` lowest)                     | ✅ (default on; `disabled` at `high` effort or below) | ✅ (always on)      | ✅      | ✅        | ❌         | ✅        | ✅              | ✅              |
| Extended thinking (explicit)  | ❌                      | ❌                                    | ❌                                                          | ❌ | ❌                  | ❌      | ❌        | ✅         | ❌        | ✅ (deprecated) | ✅ (deprecated) |
| Tool Use                      | ✅                      | ✅                                    | ✅                                                          | ✅ | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| `tool_choice: any/tool`       | ❌ (400 error)          | ❌ (400 error)                        | ❌ (400 error)                                              | ✅ (no `thinking` block) | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Vision (images)               | ✅                      | ✅                                    | ✅                                                          | ✅ | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| PDF Input                     | ✅                      | ✅                                    | ✅                                                          | ? | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Prompt Caching                | ✅ (2.5% reads)         | ✅ (5% reads)                         | ✅ (5% reads since Oct 7, 2026; 512 min)                    | ✅ (10% reads, 512 min) | ✅ (10% reads)      | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Streaming                     | ✅                      | ✅                                    | ✅                                                          | ✅ | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Batch API                     | ✅                      | ✅                                    | ✅                                                          | ✅ | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Server-Side Fallbacks         | ✅                      | ?                                    | ✅ (falls back to Sonnet 5)                                 | ❌ | ✅                  | ❌      | ❌        | ❌         | ❌        | ❌              | ❌              |
| Computer Use                  | ✅                      | ✅ (`computer_toolset_20260801` only) | ✅ (`computer_toolset_20260801` only on API / Google Cloud) | ✅ (`computer_toolset_20260801` on API / Google Cloud; `computer_20251124` on Bedrock) | ✅                  | ✅      | ✅        | ✅         | ✅        | ✅              | ✅              |
| Mid-conv tool changes         | ✅                      | ✅                                    | ✅                                                          | ? | ✅                  | ✅      | ❌        | ❌         | ✅        | ❌              | ❌              |
| Content Watermarking          | ✅                      | ?                                    | ?                                                          | ? | ❌                  | ❌      | ❌        | ❌         | ❌        | ❌              | ❌              |

`?` = not yet confirmed in the official docs for this model.

**Knowledge cutoffs:**

- Fable 5.1 / Mythos 5.1: reliable Jun 2026, training Jun 2026
- Opus 5.5: reliable Jun 2026, training Jun 2026
- Sonnet 5.5: reliable Jun 2026, training Jun 2026
- Haiku 5.5: reliable Jun 2026, training Jun 2026
- Fable 5 / Mythos 5: reliable Jan 2026, training Jan 2026
- Opus 5: reliable May 2026, training May 2026
- Sonnet 5: reliable Jan 2026, training Jan 2026
- Haiku 4.5: reliable Feb 2025, training Jul 2025
- Opus 4.8 / 4.7: reliable Jan 2026, training Jan 2026
- Opus 4.6: reliable May 2025, training Aug 2025
- Sonnet 4.6: reliable Aug 2025, training Jan 2026

**Effort defaults:**

- Opus 5.5: defaults to `medium` on the Claude API; thinking is always on (`disabled` returns 400)
- Sonnet 5.5: defaults to `high` on the Claude API; adaptive thinking on by default, lowest setting `between_tools` (`disabled` returns 400)
- Haiku 5.5: defaults to `medium` on the Claude API; adaptive thinking on by default; `thinking: {type: "disabled"}` accepted at `high` effort or below
- Opus 4.8: defaults to `high` on all surfaces
- Opus 5 / Sonnet 5 / Fable 5.1: defaults to `high` on Claude API and Claude Code
- Fable 5 / Fable 5.1: adaptive thinking is always on; `thinking: {type: "disabled"}` returns 400

## Prompt Caching Minimum Tokens

| Model                                                                                 | Minimum Cacheable Tokens |
| ------------------------------------------------------------------------------------- | ------------------------ |
| Fable 5.1, Mythos 5.1, Opus 5.5, Opus 5, Sonnet 5.5, Fable 5, Mythos 5, Haiku 5.5     | 512 tokens               |
| Opus 4.8, Sonnet 5, Sonnet 4.6, Sonnet 4.5 (deprecated); retired Opus 4.1 / 4 / Sonnet 4 | 1,024 tokens          |
| Mythos Preview, Opus 4.7                                                              | 2,048 tokens             |
| Opus 4.6, Opus 4.5, Haiku 4.5                                                         | 4,096 tokens             |

> **Correction (2026-10-10):** This table previously put Haiku 4.5, Opus 5, Fable 5 and Opus 4.7 at 1,024 tokens and all of "Opus 4.7 and earlier" at 4,096. The values above are from the prompt caching page's "Cache limitations" section (Claude API, Claude Platform on AWS, Google Cloud, Microsoft Foundry).

> **Batch API extended output:** Claude Opus 5.5, Opus 5, Sonnet 5.5, Sonnet 5, Haiku 5.5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6 support up to **300k output tokens** on the Batch API using the `output-300k-2026-03-24` beta header.

## Platform Availability

| Platform               | Notes                                                                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Claude API             | All current models                                                                                                                     |
| Amazon Bedrock         | Messages-API IDs: `anthropic.claude-fable-5-1`, `anthropic.claude-opus-5-5`, `anthropic.claude-sonnet-5-5`, `anthropic.claude-haiku-5-5` (legacy Haiku 4.5: `anthropic.claude-haiku-4-5`) |
| Google Cloud Vertex    | Use same model IDs (`claude-haiku-5-5` for Haiku 5.5); legacy Haiku 4.5 uses `claude-haiku-4-5@20251001`                               |
| Claude Platform on AWS | Same IDs as Claude API (not Bedrock-style); follows Anthropic deprecation schedule                                                     |
| Microsoft Foundry      | Current models available                                                                                                               |

## Listing Available Models

```python
import anthropic

client = anthropic.Anthropic()
models = client.models.list()
for model in models:
    print(model.id)
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();
const models = await client.models.list();
for (const model of models.data) {
  console.log(model.id);
}
```

**API endpoint:** `GET /v1/models`  
**Single model:** `GET /v1/models/{model_id}`

Each model object includes `max_input_tokens`, `max_tokens`, `lifecycle`, and a `capabilities` object. Fields added this month:

| Field                                       | Added        | Meaning                                                                                                                                                                                                                       |
| ------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `line`                                      | Oct 1, 2026  | Model line: `haiku`, `sonnet`, `opus` or `fable` (more may be added). Opus 4.5 and Opus 4.6 both report `opus`. `null` for a model in no line. Read it instead of parsing the `id`.                                               |
| `capabilities.thinking.types.disabled`      | Oct 5, 2026  | Whether the model accepts `thinking: {type: "disabled"}`. `false` exactly when that returns 400; `true` on a model without thinking. A `true` model can still reject `disabled` for another reason, such as an effort level. |
| `capabilities.server_tools`                 | Oct 6, 2026  | `web_search.supported` / `code_execution.supported`: the model accepts at least one version of that tool. `server_tools.supported` is `true` if either is. Doesn't cover other server tools such as web fetch.             |

> The top-level `capabilities.code_execution` is a different check: whether code run in the code execution tool can call your request's other tools (programmatic tool calling). For Haiku 4.5, `server_tools.code_execution.supported` is `true` while `code_execution.supported` is `false`. Organization settings (for example, an admin disabling web search) can still make a supported tool fail.

```python
model = client.models.retrieve("claude-haiku-5-5")
print(model.line)  # "haiku"
print(model.capabilities.thinking.types.disabled.supported)
print(model.capabilities.server_tools.web_search.supported)
```

```typescript
const model = await client.models.retrieve("claude-haiku-5-5");
console.log(model.line); // "haiku"
console.log(model.capabilities.thinking.types.disabled.supported);
console.log(model.capabilities.server_tools.web_search.supported);
```

> SDK attribute names above follow the API field names; the SDK changelogs could not be fetched this run, so check that your SDK version exposes them (fields arrived Oct 1–6, 2026).

## Related

- [Authentication](./authentication.md)
- [Messages API](./messages-api.md)
- [Prompt Caching](./prompt-caching.md)
- [Extended Thinking](./extended-thinking.md)
