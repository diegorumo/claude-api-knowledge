# Compliance API: Session Transcripts and Activity Feed

> **Last updated:** 2026-10-10  
> **Sources:** platform.claude.com/docs/en/manage-claude/compliance-sessions, platform.claude.com/docs/en/manage-claude/compliance-activity-feed, platform.claude.com/docs/en/manage-claude/compliance-content-data, platform.claude.com/docs/en/release-notes/overview (Sep 18, Sep 24 and Oct 8, 2026)

> **Correction (2026-09-27):** Earlier versions of this file listed wrong endpoint paths (`/v1/compliance/sessions`, `/v1/compliance/local-sessions`), wrong `product_surface` values (`claude_cowork`, `claude_for_microsoft_365`), a `Bearer` auth header, and cursor pagination for sessions. All of that has been replaced with what the official docs say. If you built against the old version of this page, re-check your integration. The 2026-09-28 revision on `main` repeated some of these errors (a `claude_for_microsoft_365` surface, a `?product_surface=` filter on a `/v1/compliance/local-sessions` path, and resolving Activity Feed names through the Admin API); those were dropped too.

**Summary:** Retrieve transcripts of sessions users run in Claude apps (Cowork, Claude Code, Claude Science, Claude for Microsoft 365, Claude in Chrome) for eDiscovery and DLP, and query the organization Activity Feed.

**Availability (session endpoints):** Claude Enterprise organizations only  
**Auth:** Compliance Access Key (`sk-ant-api01-...`) sent as `x-api-key`, plus `anthropic-version: 2023-06-01`

```
x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY
anthropic-version: 2023-06-01
```

See [Set up the Compliance API](https://platform.claude.com/docs/en/manage-claude/compliance-api-access) for key provisioning.

---

## Session endpoints overview

Session endpoints require the `read:compliance_user_data` scope. No new key, scope, setting or client update is needed as new product surfaces are added.

| Product (where it runs)                                             | Endpoint family | `product_surface`                                                                                                                                | Status                        |
| ------------------------------------------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------- |
| Cowork in Claude Desktop (user's machine)                           | Local           | `cowork`                                                                                                                                         | Stable                        |
| Claude Code (terminal, Claude Desktop, IDE extension)               | Local           | `claude_code`                                                                                                                                    | Stable                        |
| Claude for Microsoft 365 (Excel, PowerPoint, Word, Outlook add-ins) | Local           | `office_agents/excel`, `office_agents/powerpoint`, `office_agents/word`, `office_agents/outlook` (`office_agents` when the app isn't identified) | **Stable since Sep 24, 2026** |
| Claude Science desktop app                                          | Local           | `claude_science`                                                                                                                                 | Beta                          |
| Claude in Chrome (extension's built-in chat)                        | Local           | `claude_in_chrome`                                                                                                                               | Beta (added Sep 18, 2026)     |
| Cowork started on claude.ai web or mobile (runs in the cloud)       | Remote          | `cowork_remote`                                                                                                                                  | Stable                        |

Treat `product_surface` as open-ended: pass through unrecognized values.

**Not returned:** Claude Code sessions authenticated with a Console API key or run through Bedrock / Google Cloud / Foundry; Claude Code cloud sessions (claude.ai/code); local sessions in organizations with HIPAA readiness enabled; local sessions under zero data retention (excluded from lists, 404 on retrieve).

|              | Local sessions                                                               | Remote sessions                                                 |
| ------------ | ---------------------------------------------------------------------------- | --------------------------------------------------------------- |
| ID prefix    | `clls_`                                                                      | `cse_`                                                          |
| List         | `GET /v1/compliance/apps/sessions/local`                                     | `GET /v1/compliance/apps/sessions/remote`                       |
| Retrieve one | `GET /v1/compliance/apps/sessions/local/{session_id}`                        | (list only)                                                     |
| Transcript   | `GET /v1/compliance/apps/sessions/local/{session_id}/messages`               | `GET /v1/compliance/apps/sessions/remote/{session_id}/messages` |
| User filter  | None                                                                         | `user_ids[]` (1–10)                                             |
| Org filter   | None                                                                         | `organization_ids[]` (up to 500)                                |
| Time filters | `created_at.gte`, `created_at.lt`, `updated_at.gte`                          | `created_at.gte/.gt/.lt/.lte`                                   |
| Retention    | 6 years by default, or the org's finite custom conversation retention period | 6 years, unless the user deletes the session                    |
| Rate limits  | Shared Compliance API limit only                                             | Shared limit plus a separate budget for these endpoints         |

Both families are read-only (no deletion) and paginate with page tokens: pass `next_page` back as `page`, stop when `next_page` is `null` (there is no `has_more`). Lists: newest first, `limit` default 100, max 500. Messages: oldest first (`order=desc` to reverse), `limit` default 100, max 1,000; a page can end early, so keep going until `next_page` is `null`.

---

## Local sessions

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/apps/sessions/local" \
  --header "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  --header "anthropic-version: 2023-06-01" \
  --data-urlencode "created_at.gte=2026-07-01T00:00:00Z" \
  --data-urlencode "limit=100"
```

```json
{
  "data": [
    {
      "type": "compliance_local_session",
      "id": "clls_01HxKpLmNoPqRsTuVwXyZaBc",
      "organization_uuid": "9a1e0000-0000-0000-0000-000000000000",
      "workspace_id": "wrkspc_01SvYKoWVRVHoEbwESNvzYdR",
      "user": {
        "id": "user_01GpKpLmNoPqRsTuVwXyZaBc",
        "email_address": "engineer@example.com"
      },
      "product_surface": "cowork",
      "created_at": "2026-07-09T14:02:11Z",
      "updated_at": "2026-07-09T14:02:38Z"
    }
  ],
  "next_page": "page_AAEfQx7mPdLkq9Rt2VwHbZk"
}
```

Key behavior:

- `created_at.lt` must be strictly after `created_at.gte` (else 400). Timestamps are RFC 3339 with a UTC offset.
- `updated_at.gte` returns sessions with activity at or after that time. For polling, set it a few minutes **before** the previous run's start (the list value can lag), then dedupe sessions and messages on `id`.
- New sessions/messages appear after a short processing delay (typically minutes).
- `created_at` is the earliest _retained_ call, so it moves forward as calls age out of retention. Dedupe on `id`.
- Finish list walks within 24 hours. Transcript page tokens expire 24 hours after the walk's first page (400; restart without `page`).
- `user.email_address` is `null` if the user was deleted or left; on the messages endpoint it's always `null` (join on `user.id` via list/retrieve).
- Claude for Microsoft 365: deleting a conversation in the add-in is client-only; the session stays listed until retention removes it.
- 404 `Local sessions are not available.` if local sessions aren't available to your parent org; 503 while listings or content are temporarily unavailable (or while a customer-managed encryption key is unusable, for transcripts).

**Transcript** (`.../local/{session_id}/messages`): returns a `session` envelope plus `data` of `compliance_local_session_message` records with `role`, `model` (serving model on captured assistant turns, else `null`), `created_at`, `provenance`, and `content` of `text` / `tool_use` / `tool_result` blocks, each with `truncated`.

- `tool_use.input` is a **JSON-encoded string**, not an object.
- Never included: thinking blocks, the system prompt (replaced by a `[system prompt content not shown]` marker), tool definitions, MCP config. Images/PDFs appear as `[<block type> content not shown]`.
- `tool_use_input_max_bytes` / `tool_result_max_bytes` default 10,000; `-1` = server max (~1 MiB); `0` = 400. Truncated inputs are no longer valid JSON.
- `provenance` is `null` for verified captured content. Otherwise `content_unavailable` (with `reason`: `not_captured`, `client_aborted`, `retention_elapsed`, `oversize`, `cmek_key_revoked` reserved), `client_asserted`, or `synthetic_marker`. Tolerate unknown values.
- Claude Science calls MCP connectors from code in its `repl` tool, so match connector calls in the `input` string (use `tool_use_input_max_bytes=-1`). Cowork and Claude Code use `mcp__<server>__<tool>` tool names.

## Remote sessions

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/apps/sessions/remote" \
  --header "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  --header "anthropic-version: 2023-06-01" \
  --data-urlencode "created_at.gte=2026-06-01T00:00:00Z" \
  --data-urlencode "limit=100"
```

Session objects carry `id` (`cse_`), `organization_uuid`, `user` **or** `agent_id` (`cagt_`, e.g. scheduled tasks; then `started_by_user` names the initiator), `status` (`pending`, `active`, `paused`, `archived`, `failed`), `created_at`, `updated_at`, `product_surface` (currently only `cowork_remote`), `claude_project_id`. `user_ids[]` excludes agent-owned sessions. Deleted sessions are never returned.

Transcript messages carry `id` (`csev_`), `role`, `created_at` (commit time; keep returned order), `content`, `sent_by_user_id` (agent-owned sessions only), and `content_unavailable`. Thinking blocks and images aren't included. Same byte-cap parameters as local. 404 for `pending`, deleted, missing, or out-of-scope sessions.

## Example: page through local sessions (Python)

```python
import os
import requests

BASE_URL = "https://api.anthropic.com"
HEADERS = {
    "x-api-key": os.environ["ANTHROPIC_COMPLIANCE_ACCESS_KEY"],
    "anthropic-version": "2023-06-01",
}

def list_local_sessions(created_after: str) -> list[dict]:
    sessions, page = [], None
    while True:
        params = {"created_at.gte": created_after, "limit": 500}
        if page:
            params["page"] = page
        resp = requests.get(f"{BASE_URL}/v1/compliance/apps/sessions/local", headers=HEADERS, params=params)
        resp.raise_for_status()
        body = resp.json()
        sessions.extend(body["data"])
        page = body.get("next_page")
        if page is None:
            return sessions

# The list endpoint has no product_surface filter; filter client-side.
sessions = list_local_sessions("2026-09-01T00:00:00Z")
chrome = [s for s in sessions if s["product_surface"] == "claude_in_chrome"]
m365 = [s for s in sessions if (s["product_surface"] or "").startswith("office_agents")]
```

_This example is written for this knowledge base from the documented endpoints and parameters, not copied from the official page._

---

## Activity Feed

`GET /v1/compliance/activities` records authentication, chat, file, project, admin and platform activity, newest first. Requires `read:compliance_activities` on a Compliance Access Key or an Admin API key (`sk-ant-admin01-...`). Activities are queryable within 1 minute and retained for 6 years; recording starts when the Compliance API is enabled (no backfill).

- Filters: `activity_types[]`, `actor_ids[]`, `organization_ids[]` (repeat per value), `created_at.gte/.gt/.lte/.lt`.
- Cursor pagination: pass `last_id` as `after_id` for the next (older) page; stop when `has_more` is `false`. `limit` default 100, max 5,000.
- Each Activity has `id`, `created_at`, `organization_id`, `organization_uuid`, `actor` (discriminated on `actor.type`: `user_actor`, `api_actor`, `admin_api_key_actor`, `unauthenticated_user_actor`, `anthropic_actor`, `system_actor`, `scim_directory_sync_actor`), `type`, plus type-specific fields.

### File names and titles removed (Sep 24, 2026)

Activities about files, project documents and artifacts **no longer include names or titles**. The `filename` and `title` fields are always `null`, an empty string, or omitted, **including on activities recorded before Sep 24, 2026**. To get a name or title, pass the activity's `claude_file_*`, `claude_proj_doc_*` or `claude_artifact_version_*` ID to the matching metadata endpoint in [Retrieve files and artifacts](https://platform.claude.com/docs/en/manage-claude/compliance-content-data#retrieve-files-and-artifacts) using a Compliance Access Key with `read:compliance_user_data`. Lookups aren't possible after the item is deleted or when the activity has no such ID.

## Chats and Claude Docs (Oct 8, 2026, beta)

Two additions on the chat and file endpoints (source: [Retrieve and delete chats, files, and projects](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)), both in beta and using your existing Compliance Access Key:

- **Unified Claude experience chats** (Claude Enterprise): the chat endpoints ([List chats](https://platform.claude.com/docs/en/api/compliance/apps/chats/list), [Get chat messages](https://platform.claude.com/docs/en/api/compliance/apps/chats/messages/list)) now also return these chats. A chat that continues in a cloud session comes back as one chat; the work Claude does there appears in each message's `content` as `tool_use` blocks (`name`, `input`) and `tool_result` blocks (matched by `tool_use_id`). [Delete chat](https://platform.claude.com/docs/en/api/compliance/apps/chats/delete) on such a chat also deletes the cloud sessions started for it, but not sessions those sessions started.
- **Claude Docs as Word files:** Claude Docs documents are listed by [List code artifacts](https://platform.claude.com/docs/en/api/compliance/code/artifacts/list) with `artifact_type` `claude_docs` (empty `versions`, `published_version_id` `null`). Download the current text with `GET /v1/compliance/apps/code/artifacts/{artifact_id}/content?multi_file_format=zip`: a ZIP with one Word (.docx) file per tab that holds text. Comments, uploaded files and edit history aren't included.

```bash
curl "https://api.anthropic.com/v1/compliance/apps/code/artifacts/$ARTIFACT_ID/content?multi_file_format=zip" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -o claude-doc.zip
```

---

## Related

- [Set up the Compliance API](https://platform.claude.com/docs/en/manage-claude/compliance-api-access)
- [Retrieve and delete chats, files, and projects](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)
- [Compliance API errors](https://platform.claude.com/docs/en/manage-claude/compliance-errors)
- [API reference](https://platform.claude.com/docs/en/api/compliance/apps)
