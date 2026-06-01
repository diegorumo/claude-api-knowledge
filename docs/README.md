# Claude API Knowledge Base

Comprehensive reference documentation for developers building with the Claude API.

**Last full crawl:** 2026-05-30  
**Last incremental update:** 2026-06-01  
**SDK versions:** Python `anthropic` v0.105.2, TypeScript `@anthropic-ai/sdk` v0.100.1  
**Primary sources:** anthropic-sdk-python, anthropic-sdk-typescript, anthropic-cookbook

> **Coverage note:** The Anthropic documentation site (docs.anthropic.com / platform.claude.com/docs)
> was not directly accessible during crawl. Content compiled from official GitHub SDK repos and
> cookbook examples. Accuracy is high for API mechanics; check Anthropic's official docs for
> pricing tables and the most current feature status.

---

## Quick Start

- **New to Claude API?** → [Authentication](./authentication.md) + [Quick Reference](./QUICK-REFERENCE.md)
- **Building an agent?** → [Tool Use](./tool-use.md) + [Agent Patterns](./agent-patterns.md)
- **Optimizing costs?** → [Prompt Caching](./prompt-caching.md) + [Batch API](./batch-api.md)
- **Latest models?** → [Models](./MODELS.md)

---

## Documentation Index

### Core Reference

| File | Description | Last Updated |
|------|-------------|-------------|
| [MODELS.md](./MODELS.md) | Current model IDs, capabilities, context windows, pricing notes | 2026-06-01 |
| [QUICK-REFERENCE.md](./QUICK-REFERENCE.md) | Common code patterns: auth, messages, streaming, tools, caching | 2026-05-30 |
| [authentication.md](./authentication.md) | API keys, HTTP headers, SDK setup, third-party platforms | 2026-05-30 |
| [messages-api.md](./messages-api.md) | Messages endpoint: params, content blocks, response format | 2026-06-01 |
| [streaming.md](./streaming.md) | SSE events, delta types, streaming SDK helpers | 2026-05-30 |
| [rate-limits-errors.md](./rate-limits-errors.md) | Error codes, retry logic, rate limit headers | 2026-05-30 |
| [token-counting.md](./token-counting.md) | Count tokens before sending, context window management | 2026-05-30 |
| [sdks.md](./sdks.md) | Python and TypeScript SDK reference, async, pagination | 2026-05-30 |

### Features

| File | Description | Status | Last Updated |
|------|-------------|--------|-------------|
| [tool-use.md](./tool-use.md) | Function calling, agentic loop, tool choice, built-in tools | Stable | 2026-05-30 |
| [prompt-caching.md](./prompt-caching.md) | cache_control, TTL, pricing, multi-turn caching | Stable | 2026-06-01 |
| [extended-thinking.md](./extended-thinking.md) | Thinking blocks, budget_tokens, adaptive mode | Stable | 2026-05-30 |
| [vision.md](./vision.md) | Image inputs: base64, URL, formats, limits | Stable | 2026-05-30 |
| [pdf-support.md](./pdf-support.md) | PDF document inputs, Files API for PDFs | Stable | 2026-05-30 |
| [batch-api.md](./batch-api.md) | Async batch processing, results retrieval | Stable | 2026-05-30 |
| [web-search.md](./web-search.md) | Built-in web search tool, usage tracking | Stable | 2026-05-30 |
| [files-api.md](./files-api.md) | File upload, reference by ID | Beta | 2026-05-30 |
| [citations.md](./citations.md) | Inline document citations | Beta | 2026-05-30 |
| [mcp.md](./mcp.md) | Model Context Protocol server integration | Beta | 2026-05-30 |
| [computer-use.md](./computer-use.md) | GUI automation, screenshot, mouse/keyboard | Beta | 2026-05-30 |
| [managed-agents.md](./managed-agents.md) | Persistent agents, sessions, environments | Beta | 2026-05-30 |

### Guides

| File | Description | Last Updated |
|------|-------------|-------------|
| [agent-patterns.md](./agent-patterns.md) | Chaining, parallelization, routing, orchestration patterns | 2026-05-30 |
| [prompt-engineering.md](./prompt-engineering.md) | Prompting techniques, JSON output, system prompts | 2026-05-30 |
| [embeddings.md](./embeddings.md) | Voyage AI embeddings, RAG pipeline | 2026-05-30 |
| [migrations.md](./migrations.md) | Model migration, text completions → messages API | 2026-05-30 |

### Meta

| File | Description |
|------|-------------|
| [CHANGELOG.md](./CHANGELOG.md) | Crawl history and change log |

---

## Current Model IDs (Quick Reference)

```
claude-opus-4-8              # Most capable (added 2026-05-28)
claude-sonnet-4-6            # Balanced — recommended default
claude-haiku-4-5-20251001    # Fastest / lowest cost

# Preview
claude-mythos-preview        # Experimental preview model

# Previous generation (still supported)
claude-opus-4-7              # Previous Opus (added 2026-04-16)
claude-opus-4-6              # Older Opus
```

## API Base URL

```
https://api.anthropic.com
```

## Required Headers

```
x-api-key: sk-ant-...
anthropic-version: 2023-06-01
content-type: application/json
```

## Beta Features (require extra header)

| Feature | Header |
|---------|--------|
| Computer Use | `anthropic-beta: computer-use-2024-10-22` |
| MCP Client | `anthropic-beta: mcp-client-2025-04-04` |
| Citations | `anthropic-beta: citations-2024-11-06` |
| Files API | `anthropic-beta: files-api-2025-04-14` |
| Thinking Token Count | `anthropic-beta: thinking-token-count-2025-05-07` |
