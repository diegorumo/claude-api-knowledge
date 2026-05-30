# Knowledge Base Changelog

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
