# Knowledge Base Changelog

## 2026-06-01 — Incremental Update

**SDK status:** No new SDK releases since 2026-05-30 (Python v0.105.2, TypeScript v0.100.1 remain latest).

This update corrects documentation gaps from the initial crawl by inspecting SDK type definitions directly.

### Corrections & Additions

- **`README.md`:** Fixed TypeScript SDK version (was incorrectly listed as "aws-sdk v0.3.1"; correct package is `@anthropic-ai/sdk` v0.100.1)
- **`MODELS.md`:** Added missing model IDs discovered in SDK `model.py`:
  - `claude-mythos-preview` — experimental preview model (added SDK v0.90.0, 2026-04-07)
  - `claude-opus-4-7` — previous-generation Opus (added SDK v0.96.0, 2026-04-16)
  - `claude-opus-4-6` — older Opus, confirmed in SDK examples
  - Historical models: `claude-opus-4-5/4.5-20251101`, `claude-opus-4-1/4.1-20250805`, `claude-opus-4-0/4-20250514`, `claude-sonnet-4-0/4-20250514`, `claude-3-haiku-20240307`
  - Note: claude-opus-4-0 and claude-sonnet-4-0 marked deprecated since SDK v0.95.0 (2026-04-14)
- **`messages-api.md`:** Added features present in SDK but missing from initial docs:
  - `stop_details` response field — `RefusalStopDetails` with `type`, `category` (`"cyber"`, `"bio"`), `explanation`
  - `usage.output_tokens_details.thinking_tokens` — count of tokens spent on internal reasoning
  - `usage.cache_creation` breakdown — `ephemeral_5m_input_tokens` vs `ephemeral_1h_input_tokens`
  - `usage.service_tier` — `"standard"`, `"priority"`, or `"batch"`
  - New request params: `container`, `inference_geo`, `service_tier`, `output_config`
  - `output_config` parameter — `effort` levels (`"low"`, `"medium"`, `"high"`, `"xhigh"`, `"max"`) and `format` for structured JSON output (`{"type": "json_schema", "schema": {...}}`)
  - Mid-conversation system blocks — `role: "system"` messages with `type: "mid_conv_system"` content (SDK v0.105.0+)
- **`prompt-caching.md`:** Updated usage tracking to show `cache_creation.ephemeral_5m_input_tokens` and `ephemeral_1h_input_tokens` fields

---

## 2026-05-30 — Initial Full Crawl

Initial comprehensive crawl of Claude API documentation from public sources:
- GitHub: anthropics/anthropic-sdk-python (v0.105.2)
- GitHub: anthropics/anthropic-sdk-typescript (@anthropic-ai/sdk v0.100.1) [version was incorrectly recorded; corrected 2026-06-01]
- GitHub: anthropics/anthropic-cookbook
- SDK examples, changelogs, and API reference files

> **Note:** Direct access to platform.claude.com/docs and docs.anthropic.com returned HTTP 403 from
> the crawl environment. Documentation was compiled from:
> 1. Official SDK README files and API reference files (api.md, helpers.md)
> 2. Official SDK example code
> 3. Anthropic Cookbook notebooks
> 4. SDK CHANGELOG files (for recent feature additions)
> 5. Claude's built-in knowledge (knowledge cutoff August 2025, plus SDK changelog data through May 2026)

### Files Created

| File | Description |
|------|-------------|
| `README.md` | Index of all documentation files |
| `MODELS.md` | Current model IDs, capabilities, context windows |
| `QUICK-REFERENCE.md` | Common code patterns for quick lookup |
| `CHANGELOG.md` | This file |
| `authentication.md` | API keys, headers, third-party platform auth |
| `messages-api.md` | Core Messages API reference |
| `streaming.md` | SSE streaming, event types, delta types |
| `tool-use.md` | Function calling, agentic loop, built-in tools |
| `prompt-caching.md` | Cache control, TTL, pricing, best practices |
| `extended-thinking.md` | Thinking blocks, budget tokens, streaming |
| `vision.md` | Image inputs (base64, URL), supported formats |
| `pdf-support.md` | PDF document inputs |
| `batch-api.md` | Async batch processing |
| `token-counting.md` | Count tokens before sending |
| `rate-limits-errors.md` | HTTP errors, retry logic, limits |
| `sdks.md` | Python and TypeScript SDK reference |
| `agent-patterns.md` | Prompt chaining, parallelization, routing, etc. |
| `files-api.md` | File upload/reference (beta) |
| `web-search.md` | Built-in web search tool |
| `mcp.md` | Model Context Protocol integration (beta) |
| `managed-agents.md` | Persistent agents, sessions, environments (beta) |
| `prompt-engineering.md` | Prompting techniques, JSON output, patterns |
| `computer-use.md` | GUI automation (beta) |
| `embeddings.md` | Voyage AI embeddings for RAG |
| `citations.md` | Inline document citations (beta) |
| `migrations.md` | Model and API migration guides |

### SDK Features Confirmed (from CHANGELOG)

Most recent SDK releases (Python v0.100.0 – v0.105.2, May 2026):
- `claude-opus-4-8` model support (v0.105.0, 2026-05-28)
- Mid-conversation system blocks (v0.105.0)
- Usage token details (v0.105.0)
- `thinking-token-count` beta for streaming (v0.104.0)
- Self-hosted sandboxes in Managed Agents (v0.103.0)
- BetaManagedAgentsSearchResultBlock types (v0.102.0)
- Cache diagnostics beta support (v0.102.0)
- AWS client for Claude Platform (v0.101.0)
- Managed Agents multiagents, outcomes, webhooks, vault validation (v0.100.0)
