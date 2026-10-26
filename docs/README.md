# Claude API Knowledge Base

Comprehensive reference documentation for developers building with the Claude API.

**Last full crawl:** 2026-05-30  
**Last incremental update:** 2026-10-26  
**SDK versions:** Python 2.3.0, TypeScript 1.3.0 (latest on the SDK overview page, 2026-10-26; new majors Python 2.0.0 / TypeScript 1.0.0 on Oct 21, 2026 send API version `2026-10-21`; SDK changelogs not fetched)  
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

| File                                             | Description                                                                                                                                                                                                        | Last Updated |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------ |
| [MODELS.md](./MODELS.md)                         | Current model IDs, capabilities, context windows, pricing notes; Haiku 5.5 (Haiku 4.5 now legacy); Sonnet 5.5 cache-read price cut; Models API `line` / `thinking.types.disabled` / `server_tools`; Opus 5.5; Sonnet 5.5; retirement dates; Sonnet 4.5 deprecation; retired models (replacements corrected); Opus 5.5 fast mode out of research preview; Claude Models on Google Cloud (Anthropic managed) platform row | 2026-10-26   |
| [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)       | Common code patterns: auth, messages, streaming, tools, caching                                                                                                                                                    | 2026-10-10   |
| [authentication.md](./authentication.md)         | API keys (workspace/personal/service-account), HTTP headers, SDK setup, third-party platforms; API credits for Max and Team plans; `Authorization: Bearer` and one-credential rule in new SDK majors; Claude Models on Google Cloud (Anthropic managed) | 2026-10-26   |
| [messages-api.md](./messages-api.md)             | Messages endpoint: params, content blocks, response format; refusal billing; API version `2026-10-21` removals (sampling params, assistant documents, user citations); `user_profile_id` without beta header | 2026-10-26   |
| [structured-outputs.md](./structured-outputs.md) | JSON schema outputs (`output_config.format`), Pydantic/Zod helpers, strict tool use                                                                                                                                | 2026-10-10   |
| [streaming.md](./streaming.md)                   | SSE events, delta types, streaming SDK helpers                                                                                                                                                                     | 2026-06-29   |
| [rate-limits-errors.md](./rate-limits-errors.md) | Error codes, retry logic, rate limit headers                                                                                                                                                                       | 2026-05-30   |
| [token-counting.md](./token-counting.md)         | Count tokens before sending, context window management; user profile header without beta | 2026-10-26   |
| [sdks.md](./sdks.md)                             | Python and TypeScript SDK reference, async, pagination; Python v1.0.0 breaking changes; Organization API (GA under `client.organization` in Python v1.10.0 / TypeScript v0.130.0); browser/computer toolset classes (beta); version history through Python v1.10.0 / TypeScript v0.130.0; Python 2.0.0 / TypeScript 1.0.0 breaking changes (API version `2026-10-21`, Apache 2.0); `AnthropicGoogleCloud` client | 2026-10-26   |
| [api-version-2026-10-21.md](./api-version-2026-10-21.md) | API version `2026-10-21`: Messages, Admin API and Enterprise Analytics breaking changes; SDK major versions that send it (Python 2.0.0, TypeScript 1.0.0, ...); SDK and CLI code changes | 2026-10-26   |

### Features

| File                                                           | Description                                                                                                                                                                                              | Status                           | Last Updated |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------ |
| [tool-use.md](./tool-use.md)                                   | Function calling, agentic loop, tool choice, built-in tools, tool definition properties, mid-conversation tool changes, inline tool definitions (beta)                                                   | Stable                           | 2026-09-28   |
| [skills-api.md](./skills-api.md)                               | Upload and manage reusable skill packages; BetaSkill renamed BetaContainerSkill in v1.2.0+                                                                                                               | **GA**                           | 2026-08-31   |
| [programmatic-tool-calling.md](./programmatic-tool-calling.md) | Call tools from code execution sandbox, reduce round-trips and token usage                                                                                                                               | Stable                           | 2026-06-22   |
| [prompt-caching.md](./prompt-caching.md)                       | cache_control, TTL, pricing, multi-turn caching; per-model minimums; Opus 5.5 / Sonnet 5.5 5% and Fable 5.1/Mythos 5.1 2.5% cache read pricing; Haiku 5.5 tiers; cache diagnostics (GA) | Stable                           | 2026-10-10   |
| [extended-thinking.md](./extended-thinking.md)                 | Thinking blocks, budget_tokens, adaptive mode, `updates` display mode, thinking block binding (Fable 5.1), per-message effort, turn-scoped system messages                                               | Stable                           | 2026-09-07   |
| [vision.md](./vision.md)                                       | Image inputs: base64, URL, formats, limits                                                                                                                                                               | Stable                           | 2026-05-30   |
| [pdf-support.md](./pdf-support.md)                             | PDF document inputs, Files API for PDFs                                                                                                                                                                  | Stable                           | 2026-05-30   |
| [batch-api.md](./batch-api.md)                                 | Async batch processing, results retrieval                                                                                                                                                                | Stable                           | 2026-05-30   |
| [web-search.md](./web-search.md)                               | Built-in web search and web fetch tools, usage tracking; `content_too_large` error; organization web fetch on/off and domain settings | Stable                           | 2026-10-26   |
| [files-api.md](./files-api.md)                                 | File upload, reference by ID; file expiration, GA pagination; beta.files uses GA shapes in v1.2.0+                                                                                                       | **GA**                           | 2026-08-31   |
| [citations.md](./citations.md)                                 | Inline document citations; `citations` on user text blocks rejected in API version `2026-10-21` | Beta                             | 2026-10-26   |
| [mcp.md](./mcp.md)                                             | Model Context Protocol server integration + MCP Tunnels API                                                                                                                                              | Beta                             | 2026-08-03   |
| [computer-use.md](./computer-use.md)                           | GUI automation (computer_toolset_20260801 GA); browser use toolset (browser_toolset_20260801 GA); SDK toolset classes (beta) | **GA**                           | 2026-10-10   |
| [managed-agents.md](./managed-agents.md)                       | Persistent agents, sessions, threads, model effort, environments, budgets, advisor tool, inference geo pinning, web search domain config, sandbox memory; `auto` permission policy; sessions connect CLI; dynamic workflows (workflow runs); `allowed_hosts` for web tools; `web_fetch` prior-context rule; scheduled deployments (also on Claude Platform on AWS); organization web search / web fetch settings apply to sessions; Claude Models on Google Cloud (Anthropic managed) | Beta                             | 2026-10-26   |
| [compaction.md](./compaction.md)                               | Automatic and on-demand conversation compaction to extend context in long sessions; `compact-2026-01-12` / `compact-2026-09-04`                                                                          | Beta                             | 2026-09-21   |
| [compliance-api.md](./compliance-api.md)                       | Compliance API session transcripts (Cowork, Claude Code, Claude Science, Claude for Microsoft 365, Claude in Chrome), Activity Feed, unified-experience chats and Claude Docs downloads (beta); user preferences endpoint (beta); eDiscovery and DLP | Stable (Science, Chrome in beta) | 2026-10-19   |
| [fast-mode.md](./fast-mode.md) | `speed: "fast"` for up to 2.5x output speed; Opus 5.5 out of research preview (no beta header), Opus 5 / Opus 4.8 research preview; pricing, rate limit headers, fallback; Opus 5.5 on Claude Models on Google Cloud (Anthropic managed) | Stable (Opus 5.5); research preview (Opus 5, Opus 4.8) | 2026-10-26   |
| [opentelemetry.md](./opentelemetry.md) | OpenTelemetry export: SDK tracing, API telemetry destinations, content modes, token counting from spans | Beta | 2026-10-19   |
| [user-profiles.md](./user-profiles.md) | User profiles for end-customer attribution (launched Oct 20, 2026); enforcement status and actions; organization webhooks for enforcement events (beta) | Stable (webhooks in beta) | 2026-10-26   |
| [claude-tag-api.md](./claude-tag-api.md) | Claude Tag API: Slack channel / workspace settings and Access bundles for Claude Enterprise (Admin API key with `read:claude_tag` / `write:claude_tag`) | Beta | 2026-10-26   |
| [inference-hooks.md](./inference-hooks.md) | Inference hooks: allow/deny verdicts from your AI security server before inference (Claude Enterprise) | Stable | 2026-10-26   |

### Guides

| File                                             | Description                                                | Last Updated |
| ------------------------------------------------ | ---------------------------------------------------------- | ------------ |
| [agent-patterns.md](./agent-patterns.md)         | Chaining, parallelization, routing, orchestration patterns | 2026-05-30   |
| [prompt-engineering.md](./prompt-engineering.md) | Prompting techniques, JSON output, system prompts          | 2026-05-30   |
| [embeddings.md](./embeddings.md)                 | Voyage AI embeddings, RAG pipeline                         | 2026-05-30   |
| [migrations.md](./migrations.md)                 | Model migration, text completions → messages API; Haiku 4.5 → Haiku 5.5; Sonnet 4.5 → Sonnet 5.5 | 2026-10-10   |

### Meta

| File                           | Description                  |
| ------------------------------ | ---------------------------- |
| [CHANGELOG.md](./CHANGELOG.md) | Crawl history and change log |

---

## Current Model IDs (Quick Reference)

```
# Current models (as of Oct 26, 2026; unchanged since Oct 10):
claude-opus-5-5              # Recommended starting point; 1M ctx; $4/$20 MTok; always-on thinking; default effort medium; 5% cache reads
claude-fable-5-1             # Most capable; 1M ctx; $10/$50 MTok; always-on thinking; 2.5% cache reads
claude-sonnet-5-5            # Best balance; 1M ctx; $2/$10 MTok; default effort high; 5% cache reads ($0.10/MTok since Oct 7, 2026)
claude-haiku-5-5             # Fastest; 1M ctx; 128k out; $0.10/$0.50 MTok (prompts <=100k tokens), $0.50/$2.50 above; default effort medium; launched Oct 7, 2026

# Legacy / still available:
claude-opus-5                # 1M ctx; $5/$25 MTok; predecessor to Opus 5.5
claude-sonnet-5              # 1M ctx; $2/$10 MTok (price locked Aug 10, 2026); predecessor to Sonnet 5.5
claude-fable-5               # 1M ctx; $10/$50 MTok; predecessor to Fable 5.1
claude-opus-4-8              # 1M ctx; $5/$25 MTok
claude-opus-4-7              # 1M ctx; $5/$25 MTok
claude-opus-4-6              # 1M ctx; $5/$25 MTok
claude-sonnet-4-6            # 1M ctx; $3/$15 MTok
claude-opus-4-5-20251101     # 200k ctx; $5/$25 MTok; retirement not sooner than Nov 24, 2026
claude-haiku-4-5-20251001    # 200k ctx; $1/$5 MTok; predecessor to Haiku 5.5; retirement not sooner than Oct 15, 2026
claude-haiku-4-5             # Alias for claude-haiku-4-5-20251001

# Deprecated (still works; retirement scheduled):
claude-sonnet-4-5-20250929   # Deprecated Sep 30, 2026; retires Nov 30, 2026; replacement claude-sonnet-5-5

# Invitation-only:
# claude-mythos-5-1          → Project Glasswing only; same specs as Fable 5.1; 2.5% cache reads
# claude-mythos-5            → Project Glasswing only; predecessor to Mythos 5.1

# Retired (return errors):
# claude-opus-4-1 / claude-opus-4-1-20250805 → retired Aug 5, 2026
# claude-sonnet-4-20250514 / claude-opus-4-20250514 → retired Jun 15, 2026
# claude-3-haiku-20240307 → retired Apr 20, 2026 (see MODELS.md for the full list)
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

API version `2026-10-21` (Oct 21, 2026) is the latest; `2023-06-01` still works. The new SDK majors (Python 2.0.0, TypeScript 1.0.0) send `2026-10-21` and send the key as `Authorization: Bearer`. See [api-version-2026-10-21.md](./api-version-2026-10-21.md).

## Beta Features (require extra header)

| Feature                                                                                                           | Header                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Computer Use (old beta)                                                                                           | `anthropic-beta: computer-use-2024-10-22` (GA versions `computer_toolset_20260801` / `browser_toolset_20260801` need NO header) |
| MCP Client                                                                                                        | `anthropic-beta: mcp-client-2025-04-04`                                                                                         |
| Citations                                                                                                         | `anthropic-beta: citations-2024-11-06`                                                                                          |
| Files API (old format only)                                                                                       | `anthropic-beta: files-api-2025-04-14` (GA — no header needed; header keeps old response format)                                |
| Thinking Token Count                                                                                              | `anthropic-beta: thinking-token-count-2025-05-07`                                                                               |
| User Profiles                                                                                                     | `anthropic-beta: user-profiles-2026-03-24`                                                                                      |
| User Profiles (extended — `external_user_details`)                                                                | `anthropic-beta: user-profiles-2026-09-04` (user profiles launched Oct 20, 2026: no header needed; sending this or an earlier `user-profiles-` value leaves out `enforcement_status` / `enforcement_actions`) |
| Managed Agents                                                                                                    | `anthropic-beta: managed-agents-2026-04-01`                                                                                     |
| Server-Side Fallback                                                                                              | `anthropic-beta: server-side-fallback-2026-06-01`                                                                               |
| Memory Stores                                                                                                     | `anthropic-beta: agent-memory-2026-07-22`                                                                                       |
| MCP Tunnels                                                                                                       | `anthropic-beta: mcp-tunnels-2026-06-22`                                                                                        |
| Mid-Conversation Tool Changes                                                                                     | `anthropic-beta: mid-conversation-tool-changes-2026-07-01`                                                                      |
| Skills API (old format only)                                                                                      | `anthropic-beta: skills-2025-10-02` (GA — no header needed; header keeps backward compat)                                       |
| Thinking Block Binding Controls (Fable 5.1)                                                                       | `anthropic-beta: thinking-binding-controls-2026-08-01`                                                                          |
| Per-Message Effort Changes                                                                                        | `anthropic-beta: mid-conversation-output-config-2026-07-01`                                                                     |
| Turn-Scoped System Messages                                                                                       | `anthropic-beta: mid-conversation-system-clear-at-2026-08-21`                                                                   |
| Thinking Display Updates                                                                                          | `anthropic-beta: thinking-display-updates-2026-08-18`                                                                           |
| Automatic Compaction                                                                                              | `anthropic-beta: compact-2026-01-12`                                                                                            |
| On-Demand Compaction                                                                                              | `anthropic-beta: compact-2026-09-04`                                                                                            |
| Inline Tool Definitions (in mid-conversation system messages)                                                     | `anthropic-beta: inline-tools-2026-09-15`                                                                                       |
| MCP Connector (mid-conversation MCP toolsets, `mcp_tool_listing`; includes everything in `mcp-client-2025-11-20`) | `anthropic-beta: mcp-client-2026-09-15`                                                                                         |
| Fast Mode (Opus 5 / Opus 4.8 research preview)                                                                    | `anthropic-beta: fast-mode-2026-02-01` (not needed for Opus 5.5 since Oct 15, 2026)                                             |
| Telemetry Destinations (Admin API)                                                                                | `anthropic-beta: telemetry-destinations-2026-08-11`                                                                             |
| Cache Diagnostics                                                                                                 | No header needed (GA Sep 23, 2026); `cache-diagnosis-2026-04-07` still accepted                                                 |
| Organization Webhooks (Admin API)                                                                                 | `anthropic-beta: webhooks-2026-10-07`                                                                                           |
| Claude Tag API (Claude Enterprise)                                                                                | `anthropic-beta: experimental-claude-tag-2026-10-21`                                                                            |
