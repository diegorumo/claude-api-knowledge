# Knowledge Base Changelog

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
