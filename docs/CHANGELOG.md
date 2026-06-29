# Knowledge Base Changelog

## 2026-06-29 — Incremental Update

Sources: Python SDK v0.112.0 changelog, TypeScript SDK v0.106.0 changelog. Docs site (platform.claude.com/docs) returned HTTP 403/404 — data from official GitHub SDK repos.

### Changes

- **`system.message` streaming events (Python v0.112.0 / TypeScript v0.106.0)** — The SSE stream now surfaces `system.message` events emitted by Anthropic's platform infrastructure (notices, alerts). Updated `streaming.md` event types table.
- **User Profile ID support (Python v0.112.0 / TypeScript v0.106.0)** — New `user_profile_id` parameter on `messages.create()` and `messages.stream()`. Sent as the `anthropic-user-profile-id` HTTP header. Use when acting on behalf of a party other than your organization. Requires the `user-profiles` beta header. Added to `messages-api.md` request parameters table.
- **New refusal category (API-level, both SDKs)** — Additional internal refusal category added to the API. Complements the previously documented `frontier_llm` category (added v0.109.1 / v0.104.1).
- **Memory tool parent-directory fix (Python v0.112.0)** — Memory tool in Managed Agents now correctly creates parent directories with proper permissions when writing memory files. Noted in `managed-agents.md` gotchas.
- **TypeScript x-stainless-helper fix (TypeScript v0.106.0)** — Single-source for `x-stainless-helper` header, corrected append semantics, and fallback middleware tagging unified. Internal SDK hygiene.
- **SDK versions updated** — Python v0.111.0 → v0.112.0, TypeScript v0.105.0 → v0.106.0.
- **No new models** — Model list unchanged from 2026-06-22 update.

### Files Modified

| File | Change |
|------|--------|
| `streaming.md` | Added `system.message` event to SSE event types table; updated date |
| `messages-api.md` | Added `user_profile_id` parameter to request params table; updated date |
| `sdks.md` | Added Python v0.112.0 and TypeScript v0.106.0 to version history; updated date |
| `managed-agents.md` | Added memory tool parent-dir fix to gotchas; updated date |
| `README.md` | Updated last-incremental-update date; updated SDK versions; updated file last-updated dates |

---

## 2026-06-22 — Incremental Update

Sources: Python SDK v0.110.0–v0.111.0 changelog, TypeScript SDK v0.105.0 changelog, platform.claude.com/docs (programmatic-tool-calling, tool-reference pages now accessible).

### Changes

- **New feature: Programmatic Tool Calling (GA)** — `code_execution_20260120`+ lets Claude write Python code that calls your tools inside the sandbox without per-tool model round-trips. Reduces billed input tokens 20–40% for multi-tool workflows. Requires `allowed_callers: ["code_execution_20260120"]` on tools. Available on Claude API, Claude Platform on AWS, Microsoft Foundry (not Bedrock/Vertex). Created new `programmatic-tool-calling.md`.
- **New code execution tool version: `code_execution_20260521`** — Latest version; discloses per-cell time limit in tool description. `code_execution_20260120` is still active (adds programmatic tool calling). Added to `tool-use.md` version table.
- **New tool definition properties documented** — `allowed_callers`, `eager_input_streaming`, `defer_loading`, `strict`, `input_examples` are all now GA. Updated `tool-use.md` with full property table and `defer_loading` + caching interaction.
- **Updated tool type strings** — `web_search_20260318` and `web_fetch_20260318` are the newest versions (add response-inclusion control). Added to `tool-use.md` version table.
- **Retired models removed from SDKs** — Python v0.109.2 and TypeScript v0.104.2 (2026-06-15) removed retired model identifiers from SDK type definitions. Note added to `MODELS.md`.
- **Refusal-fallback middleware tagging** — Python v0.111.0 (2026-06-18): refusal-fallback middleware requests are now tagged with `fallback-refusal-middleware`. Added to `sdks.md` version table.
- **Bedrock stream event type fix** — Python v0.110.0 (2026-06-18): Bedrock stream events now preserve correct types. Added to `sdks.md` version table.
- **TypeScript lazy tool JSON streaming** — TypeScript v0.105.0 (2026-06-18): partial tool JSON input is now lazily parsed during streaming (performance improvement). Added to `sdks.md` version table.
- **SDK versions updated** — Python v0.109.1→v0.111.0, TypeScript v0.104.1→v0.105.0.

### Files Modified

| File | Change |
|------|--------|
| `programmatic-tool-calling.md` | **New file** — full reference for programmatic tool calling |
| `tool-use.md` | Updated built-in tool type strings; added Tool Definition Properties section |
| `sdks.md` | Updated version history table (Python 0.111.0, TypeScript 0.105.0) |
| `MODELS.md` | Added retired-models cleanup note; updated source date |
| `README.md` | Added programmatic-tool-calling.md to index; updated SDK versions and last-updated dates |

---

## 2026-06-15 — Incremental Update

Sources: Python SDK v0.107.1→v0.109.1 changelog, TypeScript SDK v0.102.0→v0.104.1 changelog, SDK `model_param.py` type definitions.

> **Note:** Anthropic documentation site (platform.claude.com/docs, docs.anthropic.com) continues to return HTTP 403 from this environment. Data sourced from official GitHub SDK repos.

### Changes

- **New model: `claude-fable-5`** — Completely new top-tier model family introduced in Python SDK v0.108.0 and TypeScript SDK v0.103.0 (2026-06-09). Supports **server-side fallbacks on refusal**. Context window and pricing TBD — check Anthropic's official docs. Added to `MODELS.md`, `QUICK-REFERENCE.md`, `README.md`.
- **New model: `claude-mythos-5`** — Production version of the mythos family (same SDK release as above). Supersedes `claude-mythos-preview`. Also supports server-side fallbacks on refusal. Added to all model references.
- **Server-side fallbacks on refusal** — New capability for `claude-fable-5` and `claude-mythos-5`: Anthropic's infrastructure can automatically switch models when a content policy triggers, without any client-side logic needed.
- **Client-side fallbacks middleware** — New pattern for providers that don't have server-side fallback support (Python SDK v0.108.0, TypeScript SDK v0.103.0). Documented in `sdks.md`.
- **Managed Agents deployments** — Python SDK v0.109.0 / TypeScript SDK v0.104.0 added support for deployment-based agent sessions with environment variable credentials. Added new "Deployments" section to `managed-agents.md`.
- **`frontier_llm` refusal category** — New stop/refusal reason added to the API (Python v0.109.1, TypeScript v0.104.1). Relevant when handling `stop_reason == "refusal"` responses.
- **SDK versions updated** — Python v0.107.1→v0.109.1, TypeScript v0.102.0→v0.104.1. Version history table in `sdks.md` updated.

### Files Modified

| File | Change |
|------|--------|
| `MODELS.md` | Added claude-fable-5, claude-mythos-5; updated capability table; updated source/date |
| `QUICK-REFERENCE.md` | Added claude-fable-5, claude-mythos-5 to model IDs table |
| `managed-agents.md` | Added Deployments section (v0.109.0+); updated gotchas |
| `sdks.md` | Added fallbacks middleware section; updated version history table |
| `README.md` | Updated last-updated dates; SDK versions; model ID quick reference |

---

## 2026-06-08 — Incremental Update

Sources: Python SDK v0.105.2→v0.107.1 changelog, TypeScript SDK v0.100.1→v0.102.0 changelog, SDK `model_param.py` / `messages.ts` type definitions, both SDK READMEs.

> **Note:** Anthropic documentation site (platform.claude.com/docs, docs.anthropic.com) continues to return HTTP 403 from this environment. Data sourced from official GitHub SDK repos as before.

### Changes

- **New model: `claude-opus-4-6`** — Now used as the primary Opus example model in both Python and TypeScript SDK READMEs. Added to `MODELS.md` and `QUICK-REFERENCE.md`.
- **New model: `claude-opus-4-7`** — Identified in SDK type definitions. Added to `MODELS.md` and `QUICK-REFERENCE.md`.
- **New model: `claude-mythos-preview`** — Completely new model family. Identified in SDK type definitions (June 2026); full capability details not yet published. Added to `MODELS.md` and `QUICK-REFERENCE.md` with experimental notice.
- **Deprecated: `claude-opus-4-1` / `claude-opus-4-1-20250805`** — Marked deprecated in Python SDK v0.106.0 (2026-06-05). Updated `MODELS.md` and `QUICK-REFERENCE.md`.
- **Added older models to history** — `claude-opus-4-5`, `claude-opus-4-0`, `claude-sonnet-4-0`, `claude-3-haiku-20240307` added to `MODELS.md` previous models table.
- **New built-in tool versions** — `tool-use.md` and `web-search.md` updated with latest type strings: `web_search_20260209`, `web_fetch_20260309`, `web_fetch_20260209`, `code_execution_20260120`, `memory_20250818`, `text_editor_20250728`, `search_bm25_20251119`, `search_regex_20251119`.
- **Web Fetch tool** — New built-in tool for URL retrieval added to `web-search.md`.
- **TypeScript SDK middleware** — `sdks.md` updated with middleware API example (v0.101.0+).
- **SDK versions updated** — `sdks.md` version history table updated to Python v0.107.1, TypeScript v0.102.0.
- **Python SDK Foundry improvements** — `copy()` and `with_options()` fixed; x-api-key header bug fixed for Foundry API-key auth (v0.107.1).
- **TypeScript streaming fix** — `stop_details` now carried through beta `message_delta` accumulation (v0.101.0).

### Files Modified

| File | Change |
|------|--------|
| `MODELS.md` | Added claude-opus-4-6/4-7, claude-mythos-preview; deprecated claude-opus-4-1; expanded history table |
| `QUICK-REFERENCE.md` | Updated model IDs table |
| `tool-use.md` | Updated built-in tools section with latest type versions |
| `web-search.md` | Updated tool type strings; added web fetch tool section |
| `sdks.md` | Added middleware example; updated version history table |
| `README.md` | Updated last-updated dates; updated SDK versions |

---

## 2026-05-30 — Initial Full Crawl

Initial comprehensive crawl of Claude API documentation from public sources:
- GitHub: anthropics/anthropic-sdk-python (v0.105.2)
- GitHub: anthropics/anthropic-sdk-typescript (aws-sdk v0.3.1)
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
