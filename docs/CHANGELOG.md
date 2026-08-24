# Knowledge Base Changelog

## 2026-08-24 — Incremental Update

Sources: platform.claude.com/docs/en/release-notes/overview (Aug 18–20, 2026), Python SDK CHANGELOG v0.123.0–v1.0.0 (2026-08-18–20), TypeScript SDK CHANGELOG v0.118.0–v0.120.0 (2026-08-18–19).

### Changes

- **Python SDK v1.0.0 (2026-08-20) — MAJOR BREAKING CHANGES** — The HTTP layer moves from `httpx` to `httpx2` (a maintained, API-compatible fork). Build custom `http_client`, `Timeout`, and transport objects from `httpx2`; call `httpx2.alias_httpx()` if you rely on tracing or mocking libraries that patch `httpx`. Additional breaking changes: Python 3.10+ required (was 3.9+); legacy Text Completions API removed; `temperature`, `top_p`, `top_k` on Messages methods removed; tool runner's client-side `compaction_control` removed; async `.with_raw_response` results now need `await response.parse()`; `AnthropicBedrock` raises error when no AWS region configured (previously defaulted to `us-east-1`). Non-breaking: fix output_format warning on parse/stream/tool_runner; restore streaming event imports; use adaptive thinking in examples. See [MIGRATION.md](https://github.com/anthropics/anthropic-sdk-python/blob/main/MIGRATION.md).
- **Files API out of beta — GA (2026-08-19)** — `/v1/files` endpoints and Messages API requests referencing uploaded files no longer require the `files-api-2025-04-14` beta header. GA response format adds file expiration (`expires_in_seconds` on upload, `expires_at` on file object) and improved pagination (`page`/`next_page` cursors, `ids[]` filter on list). Requests that still send the beta header continue to work with the previous response format (no expiration fields).
- **Skills API out of beta — GA (2026-08-19)** — `/v1/skills` endpoints and Messages API requests loading skills via the `container` parameter no longer require the `skills-2025-10-02` beta header. Requests that still send the header continue to work unchanged.
- **Computer use toolset GA (2026-08-19)** — `computer_toolset_20260801` is out of beta. No beta header required. New capabilities: batch actions (multiple actions per turn), zoom enabled by default, per-member configuration via `configs`. Previous beta versions (`computer_20241022`, `computer_20251124`) remain available under their original beta headers. See [Migrate from `computer_20251124`](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124).
- **Browser use toolset launched — GA (2026-08-19)** — New `browser_toolset_20260801` client toolset. Operates within a browser viewport rather than the full desktop. Reads the page's accessibility tree and element structure directly; adds element references, form input, tab management, download reporting, and opt-in file upload on top of screenshot-and-click control. No beta header required.
- **Both new toolsets available on** Claude Fable 5, Claude Mythos 5, Claude Opus 5, Claude Sonnet 5, Claude Opus 4.8.
- **Admin API user-management endpoints out of beta (2026-08-19)** — Group and custom-role endpoints no longer require the `ce-user-management-2026-07-13` header. Requests that still send it are accepted unchanged.
- **Managed Agents web search/fetch domain restrictions (2026-08-19 / Python v0.125.0, TypeScript v0.120.0)** — `web_search` and `web_fetch` tools in `agent_toolset_20260401` now accept `allowed_domains` or `blocked_domains` in their `configs` entries. `web_fetch` also accepts `max_content_tokens`; `web_search` accepts `user_location`. Typed per-tool types added in SDKs.
- **Self-hosted sandbox memory stores (2026-08-19 / Python v0.125.0, TypeScript v0.120.0)** — Sessions running in a self-hosted sandbox can now attach memory stores. SDK workers download the store to its `mount_path` in the sandbox at session start and sync changes back after each turn.
- **Console: Workbench → Playground (2026-08-18)** — Workbench is now **Playground** at `platform.claude.com/playground`. Supports every Messages API parameter, includes feature-demo templates, shows the full SDK request and API response. The legacy Workbench (which was sunsetted on Aug 17) and its experimental prompt tools APIs (`/v1/experimental/generate_prompt` etc.) are fully retired.
- **Python v0.123.0 / TypeScript v0.118.0 (2026-08-18)** — Additions to files and memory stores APIs; updates to skill, files, and user profiles; workspace ID helpers in response headers; bug fixes: remove unsupported `mid_conv_system` content block, compute platform headers without subprocess, export `ServiceUnavailableError`/`DeadlineExceededError`, retry tool-result sends for at least lease TTL, run synchronous session tools in worker thread (Python); TypeScript: bump zod to 4.4.3, warn about blocking tool bodies stalling worker heartbeat.

### Files Modified

| File | Change |
|------|--------|
| `computer-use.md` | **Major rewrite**: added Computer Use Toolset (GA) section for `computer_toolset_20260801`; added Browser Use Toolset (GA) section for `browser_toolset_20260801`; added comparison table; updated status and models; updated date |
| `files-api.md` | Updated status to GA; added GA migration note (expiration, new pagination); updated code examples to use `client.files.*`; added file expiration section; updated gotchas; updated date |
| `skills-api.md` | Updated status to GA; updated code examples to use `client.skills.*`; updated gotchas; updated date |
| `managed-agents.md` | Added "Web Search / Web Fetch Domain Restrictions" section; added "Self-Hosted Sandbox Memory Stores" section; updated gotchas with v0.123.0–v0.125.0 notes; updated SDK changelog version range; updated date |
| `sdks.md` | Added Python v1.0.0 breaking changes table; added Python v0.123.0–v1.0.0 to version history; added TypeScript v0.118.0–v0.120.0 to version history; updated Python requires to 3.10+; updated date |
| `README.md` | Updated last-incremental-update date (2026-08-24); updated SDK versions (Python v1.0.0, TypeScript v0.120.0); updated file status and last-updated dates; updated beta headers table to reflect Files API and Skills API GA; added computer_toolset_20260801/browser_toolset_20260801 note |

---

## 2026-08-17 — Incremental Update

Sources: Python SDK v0.122.0 changelog (2026-08-13), TypeScript SDK v0.117.0/v0.117.1 changelog (2026-08-13), platform.claude.com/docs/en/release-notes/overview, platform.claude.com/docs/en/about-claude/models/overview, platform.claude.com/docs/en/managed-agents/agent-setup.

### Changes

- **Python SDK v0.122.0 (2026-08-13)** — Multiple features and bug fixes:
  - `output_behavior` parameter for dream creation: `"create_new"` (default, creates a new memory store) or `"update_input"` (overwrites the input store in place)
  - Fixed SigV4 signing in async Bedrock clients (was running on the event loop)
  - Exposed `beta.messages.parse`, `stream`, and `tool_runner` in Bedrock client
  - Exposed beta methods in Vertex client
  - Added missing models to client
  - Maintained token exchange binding across `copy()` operations
  - PathLike file contents now handled in file tuples
  - Empty `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN` now treated as unset
  - Fixed malformed tool input JSON error context in streaming
  - Applied all `message_delta` fields during message accumulation
  - Emitted `input_json` events for server tool use blocks
  - Preserved omitted content block fields in accumulated messages
  - Ran request transformation once in `messages.stream()` (not twice)
  - Silenced pydantic warnings on `message_stop` events
  - Rejected symlink loops and special skill-archive members in tool paths
  - Inference geo (`inference_geo`) pinning support in Managed Agents model config
- **TypeScript SDK v0.117.0 (2026-08-13)** — Matching features and fixes:
  - `output_behavior` parameter for dream creation (same as Python)
  - Fixed build process to include dotfiles during distribution flattening
  - Added missing model references to client
  - Enhanced message timeout handling for non-streaming requests
  - Corrected `message_delta` field accumulation in streaming
  - Tool-runner now forwards response container IDs between requests
  - Aligned path resolution and skill-archive handling with Python SDK
  - Switched from Yarn to pnpm
  - Updated documentation for optional user profile names in resold profiles
- **TypeScript SDK v0.117.1 (2026-08-13)** — CI improvements, no API changes
- **`anthropic-workspace-id` response header (Aug 11, 2026)** — All Claude API responses now include an `anthropic-workspace-id` response header carrying the `wrkspc_`-prefixed workspace ID. Useful for tracking which workspace a request's API key resolved to, matching against Usage/Cost API reports, and debugging multi-workspace setups. The header is absent on Admin API requests and failed auth (401) responses. Added note to `authentication.md`.
- **Compliance API: local session transcripts (Aug 11, 2026, Enterprise beta)** — `GET /v1/compliance/apps/sessions/local` lists local Cowork and Claude Code sessions; `GET /v1/compliance/apps/sessions/local/{id}` retrieves metadata; `GET /v1/compliance/apps/sessions/local/{id}/messages` returns transcripts. Uses existing Compliance Access Key with `read:compliance_user_data` scope.
- **Claude Sonnet 5 pricing confirmed at $2/$10 MTok (Aug 10, 2026)** — The previously scheduled increase to $3/$15 on Sept 1, 2026 will not occur. Price is now standard. Updated `MODELS.md`.
- **Official model specs now available** — Confirmed from Anthropic docs (previously TBD): Fable 5/Opus 5/Sonnet 5 all have 1M token context windows and 128k max output; Haiku 4.5 has 200k context and 64k max output. Pricing confirmed. Extended thinking marked as deprecated on Opus 4.6 and Sonnet 4.6. Updated `MODELS.md` comprehensively.
- **Inference geo pinning in Managed Agents** — `inference_geo: "us" | "global"` can be set in the `model` object when creating an agent. Validates against workspace `allowed_inference_geos` on every turn. Can be overridden per-session. Added section to `managed-agents.md`.
- **No new Claude model IDs** in this SDK release cycle.

### Files Modified

| File | Change |
|------|--------|
| `MODELS.md` | Major update: filled in all TBD fields with confirmed specs; added official pricing; capability table rewritten with correct thinking/effort info; added platform availability table; updated date and source versions |
| `managed-agents.md` | Added "Pinned Inference Geo" section; added Dreams `output_behavior` subsection; updated gotchas; updated date and SDK changelog line |
| `sdks.md` | Added Python v0.122.0 and TypeScript v0.117.0/v0.117.1 to version history table; updated date |
| `README.md` | Updated last-incremental-update date (2026-08-17); updated SDK versions (Python v0.122.0, TypeScript v0.117.1); updated file last-updated dates; added `anthropic-workspace-id` to response headers section |

---

## 2026-08-10 — Incremental Update

Sources: Python SDK v0.121.0 changelog (2026-08-07), TypeScript SDK v0.116.0 changelog (2026-08-07), GitHub SDK type definitions. Docs site (platform.claude.com/docs) returned HTTP 404.

### Changes

- **Skills API — new beta resource (Python v0.121.0 / TypeScript v0.116.0, 2026-08-07)** — New `client.beta.skills.*` resource for uploading, listing, retrieving, and deleting reusable skill packages. A skill is a directory of files that must include a `SKILL.md` descriptor at the root; uploaded via multipart form. Response object fields: `id` (`skill_01…`), `type: "skill"`, `source` (`"custom" | "anthropic"`), `display_title`, `latest_version`, `created_at`, `updated_at`. Skills are referenced in Managed Agents sessions via `skills: [{ type: "anthropic"|"custom", skill_id, version }]`. Anthropic also maintains curated skills (e.g. `"xlsx"`) accessible via `type: "anthropic"`. Created `skills-api.md`; updated `managed-agents.md`.
- **Session Budgets (Python v0.121.0 / TypeScript v0.116.0, 2026-08-07)** — Sessions now accept a `budget_limit` parameter with `type: "limit"` and `max_list_cost` (`BetaMonetaryAmount` with `amount` + `currency`). When the session's accumulated list cost reaches the limit, no further model requests are issued. Added "Session Budgets" section to `managed-agents.md`.
- **Advisor Tool — `advisor_20260301` (Python v0.121.0 / TypeScript v0.116.0, 2026-08-07)** — New built-in tool type `"advisor_20260301"` (name must be `"advisor"`) lets the session's primary thread consult a second Claude model mid-turn. The advisor model is specified via the `model` field; optional `max_tokens` and `max_uses` cap its usage. Can also be configured via the session roster (`advisor: { type: "advisor", model: "..." }`). Max 1 advisor per session; reserved name `anthropic.advisor`. Result appears as `BetaAdvisorResultBlock` / `BetaAdvisorRedactedResultBlock` in the primary thread. Added "Advisor Tool" section to `managed-agents.md`.
- **GitHub Repository Skills Auto-Loading (Python v0.121.0 / TypeScript v0.116.0, 2026-08-07)** — Sessions can reference a GitHub repository resource (`BetaManagedAgentsGitHubRepositoryResourceConfig`) to auto-load skills stored in the repo. Added to `managed-agents.md` and `skills-api.md`.
- **Official beta header: `mid-conversation-tool-changes-2026-07-01`** — The tool_addition/tool_removal blocks (documented 2026-07-27) now have an official beta header. SDK adds it automatically when the feature is used; for raw HTTP add `anthropic-beta: mid-conversation-tool-changes-2026-07-01`. Updated `tool-use.md` and `README.md` beta headers table.
- **Retired models removed from SDK type definitions (Python v0.121.0 / TypeScript v0.116.0, 2026-08-07)** — `claude-opus-4-1` and `claude-opus-4-1-20250805` formally removed. Migrate to `claude-opus-4-6` or newer.
- **TypeScript: bash errors matchable by class (TypeScript v0.116.0, 2026-08-07)** — Bash timeout and abort errors in the agent toolset now expose their class for pattern-matched error handling. Added to `sdks.md`.
- **No new models** — No new Claude model IDs in this SDK release cycle.

### Files Modified

| File | Change |
|------|--------|
| `skills-api.md` | **New file** — full reference for Skills API (create, retrieve, list, delete; skill object; session integration; GitHub auto-loading) |
| `managed-agents.md` | Added "Session Budgets" section; added "Advisor Tool" section; added "Skills in Sessions" section; added "GitHub Repository Resource" section; updated gotchas; updated date and SDK changelog line |
| `tool-use.md` | Added official `mid-conversation-tool-changes-2026-07-01` beta header note to Mid-Conversation Tool Changes section; updated date |
| `sdks.md` | Added Python v0.121.0 and TypeScript v0.116.0 to version history table |
| `README.md` | Updated last-incremental-update date (2026-08-10); updated SDK versions (Python v0.121.0, TypeScript v0.116.0); added `skills-api.md` to Features index; updated `managed-agents.md`, `tool-use.md`, `sdks.md` last-updated dates; added `mid-conversation-tool-changes-2026-07-01` to beta headers table |

---

## 2026-08-03 — Incremental Update

Sources: Python SDK v0.120.1–v0.120.2 changelog (2026-07-28), PyPI release history. TypeScript SDK remains at v0.115.0 (no new release). Docs site (platform.claude.com/docs) returned HTTP 404.

### Changes

- **Python SDK v0.120.2 (2026-07-28) — MCP SDK v2 dual-compatibility:** The `anthropic[mcp]` extra now supports MCP SDK v1 and v2 simultaneously, superseding the pin from v0.120.1. Developers using the Python SDK's MCP integration can upgrade to MCP SDK v2 without being blocked. Updated `mcp.md` gotchas; updated `sdks.md` version history.
- **Python SDK v0.120.1 (2026-07-28) — MCP extra version pin:** Intermediate release that pinned `mcp<2` to prevent accidental breakage from MCP SDK v2. Superseded by v0.120.2. Added to `sdks.md` version history for completeness.
- **No new models or API changes** — TypeScript SDK unchanged at v0.115.0; no model additions or deprecations this cycle.

### Files Modified

| File | Change |
|------|--------|
| `sdks.md` | Added Python v0.120.1 and v0.120.2 to version history table; updated date |
| `mcp.md` | Added MCP SDK v2 compatibility note to Gotchas section; updated date |
| `README.md` | Updated last-incremental-update date (2026-08-03); updated Python SDK version (v0.120.2); updated file last-updated dates |

---

## 2026-07-27 — Incremental Update

Sources: Python SDK v0.118.0–v0.120.0 changelog (2026-07-22–24), TypeScript SDK v0.112.4–v0.115.0 changelog (2026-07-20–24), GitHub code search on both SDK repos. Docs site (platform.claude.com/docs) not verified this cycle.

### Changes

- **New model: `claude-opus-5` (Python v0.120.0 / TypeScript v0.115.0, 2026-07-24)** — New Opus-class model added. Context window and pricing TBD — check Anthropic's official docs. Added to `MODELS.md`, `QUICK-REFERENCE.md`, `README.md`.
- **New stop reason: `model_context_window_exceeded` (Python v0.119.0 / TypeScript v0.114.0, 2026-07-23)** — Added to `StopReason` and `BetaStopReason` type aliases. Returned when the conversation exceeds the model's context window. Handle by trimming old messages before retrying. Added to `messages-api.md` stop reasons table.
- **Tool addition/removal blocks and `tool_change` events (Python v0.120.0 / TypeScript v0.115.0, 2026-07-24)** — Two new content block types: `tool_addition` (expose a declared tool mid-conversation) and `tool_removal` (withdraw a tool mid-conversation). Reference a tool via `BetaToolChangeToolReferenceParam` (type `"tool"`), `BetaToolChangeMCPToolReferenceParam` (MCP tool), or `BetaToolChangeMCPToolsetReferenceParam` (MCP toolset). A `tool_change` streaming event mirrors these transitions. Added full section to `tool-use.md`.
- **Managed Agents model effort (Python v0.118.0 / TypeScript v0.113.0, 2026-07-22)** — `BetaManagedAgentsModelConfig` now has an `effort` field (five levels: low/medium/high/xhigh/max) that sets `output_config.effort` on every turn. Also exposes a `speed` field (`"standard"` | `"fast"`). Added section to `managed-agents.md`.
- **Initial session events (Python v0.118.0 / TypeScript v0.113.0, 2026-07-22)** — `sessions.create()` now accepts `initial_events: []` — up to 50 `user.message` or `user.define_outcome` events processed at session creation, eliminating the first `events.send()` round-trip. Added section to `managed-agents.md`.
- **Threads delta streaming (Python v0.118.0 / TypeScript v0.113.0, 2026-07-22)** — New `client.beta.sessions.threads.*` sub-resource (retrieve, list, archive threads; list/stream per-thread events). Endpoint: `GET /v1/sessions/{id}/threads/{thread_id}/stream`. `event_deltas=True` enables incremental delta streaming per thread. Full type inventory documented. Added section to `managed-agents.md`.
- **Expanded `fallback_credit_token` types (Python v0.120.0 / TypeScript v0.115.0, 2026-07-24)** — `fallback_credit_token` now accepts either a bare `str` (existing behavior, mode `"strict"`) or a `BetaFallbackCreditTokenParam` object with `token` + `mode` (`"strict"` | `"best_effort"`). `"best_effort"` serves the retry even if the token redemption fails. Requires `anthropic-beta: fallback-credit-2026-07-01` header for object form. Server-side fallbacks also now accept the string `"default"` for built-in fallback config. Updated `sdks.md`.
- **Binary file fix in agent toolset (Python v0.119.0, 2026-07-23)** — Agent toolset read/edit operations now correctly handle binary files. Added to `managed-agents.md` gotchas.
- **New refusal category (TypeScript v0.112.5, 2026-07-21)** — Additional internal refusal category added; no consumer-visible type changes.
- **AWS `withOptions()` fix (TypeScript v0.112.4, 2026-07-20)** — AWS options and auth mode are now correctly preserved across `withOptions()` calls.

### Files Modified

| File | Change |
|------|--------|
| `MODELS.md` | Added `claude-opus-5` to current models table and capabilities table; updated source version and date |
| `QUICK-REFERENCE.md` | Added `claude-opus-5` to model IDs table; updated date |
| `tool-use.md` | Added "Mid-Conversation Tool Changes" section (tool_addition/tool_removal blocks, tool_change events, reference type table); updated date |
| `managed-agents.md` | Added "Model Effort" section; added "Initial Session Events" section; added "Threads Delta Streaming" section; updated gotchas; updated date and SDK changelog line |
| `messages-api.md` | Added `pause_turn`, `refusal`, and `model_context_window_exceeded` to stop reasons table; updated date |
| `sdks.md` | Added Python v0.118.0–v0.120.0 and TypeScript v0.112.4–v0.115.0 to version history table; updated date |
| `README.md` | Updated last-incremental-update date (2026-07-27); updated SDK versions (Python v0.120.0, TypeScript v0.115.0); added `claude-opus-5` to model quick reference; updated file last-updated dates |

---

## 2026-07-20 — Incremental Update

Sources: Python SDK v0.117.0 changelog (2026-07-16), TypeScript SDK v0.112.0–v0.112.3 changelog (2026-07-16–17), GitHub code search on both SDK repos. Docs site (platform.claude.com/docs) returned HTTP 404.

### Changes

- **New API: MCP Tunnels (Python v0.117.0 / TypeScript v0.112.0, 2026-07-16)** — New beta resource group for provisioning secure tunnels that expose local MCP servers to Claude's infrastructure. Beta header: `mcp-tunnels-2026-06-22`. Anthropic allocates a globally unique `domain` (hostname) per tunnel; the operator runs a connector using the `tunnel_token`. TLS is enforced via registered CA certificates. Tunnel endpoints: `POST/GET/archive /v1/tunnels`, `reveal_token`, `rotate_token`. Certificate sub-resource: `POST/GET/archive /v1/tunnels/{id}/certificates`. Types: `BetaTunnel` (id `tnl_…`, domain, display_name, created_at, archived_at), `BetaTunnelToken` (id, tunnel_token), `BetaTunnelCertificate` (id `tcrt_…`, fingerprint, tunnel_id, expires_at). Max 2 non-archived certs per tunnel. A tunnel rejects all MCP traffic until at least one cert is registered. Added comprehensive MCP Tunnels section to `mcp.md`.
- **Dreams API now in Python (Python v0.117.0, 2026-07-16)** — `client.beta.dreams.*` is now available in the Python SDK (was TypeScript-only since v0.111.0). All 5 endpoints supported: `create`, `retrieve`, `list`, `archive`, `cancel`. Python's `list()` additionally accepts `created_at_gt` and `created_at_lt` datetime filters not yet in TypeScript. Requires `agent-memory-2026-07-22` beta header. Updated `managed-agents.md` Dreams section.
- **SecretStr credential security (Python v0.117.0)** — Sensitive credential fields (e.g., `tunnel_token`) are now wrapped in `SecretStr` to prevent accidental exposure in Python tracebacks / frame locals. Noted in `mcp.md` tunnel gotchas.
- **TypeScript v0.112.1–v0.112.3 (2026-07-17)** — Documentation updates only; no API or behavior changes.

### Files Modified

| File | Change |
|------|--------|
| `mcp.md` | Added full MCP Tunnels section: concepts, tunnel endpoints, certificate endpoints, Python + TypeScript examples, type tables, gotchas; updated date and status line |
| `managed-agents.md` | Updated Dreams section header to reflect Python parity (v0.117.0); added Python `list()` example with `created_at_gt` filter; updated gotchas; updated SDK changelog line; updated date |
| `sdks.md` | Added Python v0.117.0 and TypeScript v0.112.0–v0.112.3 to version history table; updated date |
| `README.md` | Updated last-incremental-update date (2026-07-20); updated SDK versions (Python v0.117.0, TypeScript v0.112.3); updated mcp.md and managed-agents.md and sdks.md last-updated dates; added `mcp-tunnels-2026-06-22` to beta headers table |

---

## 2026-07-13 — Incremental Update

Sources: TypeScript SDK v0.111.0 changelog (2026-07-10), GitHub code search on anthropics/anthropic-sdk-typescript. Python SDK remains at v0.116.0 (no new release). Docs site (platform.claude.com/docs) returned HTTP 404.

### Changes

- **New API: Dreams (TypeScript v0.111.0, 2026-07-10)** — New beta endpoint for asynchronous memory-consolidation jobs. A Dream reads a memory store plus a set of session transcripts and writes consolidated memories into a new output memory store. Endpoints: `POST /v1/dreams`, `GET /v1/dreams/{dream_id}`, `GET /v1/dreams`, `POST /v1/dreams/{dream_id}/archive`, `POST /v1/dreams/{dream_id}/cancel`. Status lifecycle: `pending → running → completed | failed | canceled`. Requires `agent-memory-2026-07-22` beta header. Python SDK support not yet available. Added "Dreams API" section to `managed-agents.md`.
- **Session tool call permissions — `evaluated_permission` (TypeScript v0.111.0, 2026-07-10)** — `agent.tool_use` and `agent.mcp_tool_use` session events now carry an `evaluated_permission` field (`'allow' | 'ask' | 'deny'`). `'ask'` holds tool execution until a `user.tool_confirmation` event arrives with `result: 'allow' | 'deny'`. `'deny'` yields the call to consumers but does not execute it. The `SessionToolRunner` helper handles this flow automatically, including idle bounding: it stops after `maxIdleMs` (default 60 s) once `stop_reason: end_turn` is reached, with the countdown paused while confirmations are outstanding. Added "Session Tool Call Permissions" section to `managed-agents.md`.
- **No new Python SDK version** — Python SDK remains at v0.116.0 (released 2026-07-02). Dreams and `evaluated_permission` are TypeScript-only for now.

### Files Modified

| File | Change |
|------|--------|
| `managed-agents.md` | Added "Dreams API" section (endpoints, types, code example, gotchas); added "Session Tool Call Permissions" section (`evaluated_permission` flow, `user.tool_confirmation`, `SessionToolRunner` idle bounding); updated gotchas list; updated date |
| `sdks.md` | Added TypeScript v0.111.0 to version history table; updated date |
| `README.md` | Updated last-incremental-update date (2026-07-13); updated TypeScript SDK version (v0.111.0); updated file last-updated dates for modified files |

---

## 2026-07-06 — Incremental Update

Sources: Python SDK v0.113.0–v0.116.0 changelog, TypeScript SDK v0.107.0–v0.110.0 changelog. Docs site (platform.claude.com/docs) returned HTTP 404 — data from official GitHub SDK repos.

### Changes

- **New model: `claude-sonnet-5` (Python v0.114.0 / TypeScript v0.108.0, 2026-06-30)** — New Sonnet generation added to SDK. Context window and pricing TBD — check Anthropic's official docs. Added to `MODELS.md`, `QUICK-REFERENCE.md`, `README.md`.
- **Managed Agents: event delta streaming (Python v0.115.0 / TypeScript v0.109.0, 2026-06-30)** — Session events now emit delta variants (e.g., `agent.message.delta`) for real-time incremental content delivery. Added new "Event Delta Streaming" section to `managed-agents.md`.
- **Managed Agents: agent overrides (v0.115.0 / v0.109.0, 2026-06-30)** — Sessions can override agent parameters (model, system prompt, max tokens) per-session without modifying the agent definition. Added "Agent Overrides" section to `managed-agents.md`.
- **Managed Agents: reverse pagination (v0.115.0 / v0.109.0, 2026-06-30)** — List endpoints now accept `order: "desc"` for newest-first pagination. Added "Reverse Pagination" section to `managed-agents.md`.
- **Managed Agents: vault credential injection scoping (v0.115.0 / v0.109.0, 2026-06-30)** — Vault credentials can now be scoped to specific MCP servers via `credential_scope`. Added "Vault Credential Injection Scoping" section to `managed-agents.md`.
- **Managed Agents: webhook events for agents and deployments (v0.115.0 / v0.109.0, 2026-06-30)** — Webhook callbacks can now be configured on both agent definitions and deployments. Added "Agent and Deployment Webhook Events" section to `managed-agents.md`.
- **New beta header: `agent-memory-2026-07-22` (Python v0.116.0 / TypeScript v0.110.0, 2026-07-02)** — Memory Stores API operations now require this beta header (SDK adds it automatically). Updated `managed-agents.md` Memory Stores section and `README.md` beta headers table.
- **Tool type strings updated to `20260318` (Python v0.113.0 / TypeScript v0.107.0, 2026-06-29)** — `web_search_20260318` and `web_fetch_20260318` are now the current versions. Updated `web-search.md` with correct latest type strings.
- **`user_profile_id` in `count_tokens` (v0.113.0 / v0.107.0, 2026-06-29)** — `count_tokens` endpoint now accepts `user_profile_id`; SDK sends it as `anthropic-user-profile-id` header. Added note to `token-counting.md`.
- **Async `count_tokens` bug fix (Python v0.113.0)** — Missing merge block for `output_format`/`output_config` in async Python `count_tokens` fixed. Added gotcha to `token-counting.md`.
- **TypeScript `BatchCreateParams.Request.params` restored (TypeScript v0.107.0)** — Type that was accidentally dropped during a codegen merge is restored. Noted in `sdks.md` version history.
- **SDK type cleanup (Python v0.115.1 / TypeScript v0.109.1, 2026-07-01)** — Nonfunctional types removed from both SDKs.
- **SDK versions updated** — Python v0.112.0 → v0.116.0, TypeScript v0.106.0 → v0.110.0.

### Files Modified

| File | Change |
|------|--------|
| `MODELS.md` | Added `claude-sonnet-5` to current models table and capability table; updated date and source version |
| `QUICK-REFERENCE.md` | Added `claude-sonnet-5` to model IDs table; updated date |
| `managed-agents.md` | Added sections: Event Delta Streaming, Agent Overrides, Reverse Pagination, Vault Credential Injection Scoping, Agent and Deployment Webhook Events, Memory Stores beta header; updated gotchas; updated date |
| `sdks.md` | Added Python v0.113.0–v0.116.0 and TypeScript v0.107.0–v0.110.0 to version history; updated date |
| `token-counting.md` | Added `user_profile_id` in token counting section; added async bug fix note; updated date |
| `web-search.md` | Updated tool type strings to `web_search_20260318` / `web_fetch_20260318`; updated date |
| `README.md` | Updated last-incremental-update date; updated SDK versions; added `claude-sonnet-5` to model quick reference; expanded beta headers table; updated file last-updated dates |

---

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
