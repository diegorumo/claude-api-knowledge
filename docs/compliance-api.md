# Compliance API — Session Transcripts

**Summary:** Retrieve transcripts of sessions users run in Claude apps (Cowork, Claude Code, Claude Science, Claude for Microsoft 365, Claude in Chrome) for eDiscovery and DLP enforcement.

**Availability:** Claude Enterprise organizations only  
**Auth:** Compliance Access Key with `read:compliance_user_data` scope

---

## Overview

The Compliance API session endpoints expose conversation transcripts from Claude apps running in your Enterprise organization. Each session is one conversation; its transcript is the ordered sequence of user prompts, assistant responses, and tool calls/results.

**Supported surfaces:**

| `product_surface` | Status |
|-------------------|--------|
| `claude_cowork` | Stable |
| `claude_code` | Stable |
| `claude_science` | Beta |
| `claude_for_microsoft_365` | Beta |
| `claude_in_chrome` | Beta (added Sep 18, 2026) |

Remote sessions (Managed Agents / cloud) and local sessions (Claude Code, Claude in Chrome) use separate endpoints.

---

## Authentication

All Compliance API requests use a **Compliance Access Key** (separate from API keys), with the `read:compliance_user_data` scope. No new key, scope, or client update is needed for the new Chrome sessions endpoint.

```
Authorization: Bearer <compliance-access-key>
anthropic-version: 2023-06-01
```

See [Set up the Compliance API](https://platform.claude.com/docs/en/manage-claude/compliance-api-access) for key provisioning.

---

## Remote sessions (cloud)

Remote sessions are sessions running in the cloud — Cowork, Managed Agents, Claude Science.

### List remote sessions

```
GET /v1/compliance/sessions
```

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `user_id` | string | Filter to one user's sessions (get IDs from List Organization Users) |
| `product_surface` | string | Filter by surface (e.g. `claude_cowork`) |
| `after` | string | Pagination cursor |
| `before` | string | Pagination cursor |
| `limit` | integer | Max results per page |

**Response:**

```json
{
  "sessions": [
    {
      "id": "sess_abc123",
      "user_id": "user_xyz",
      "product_surface": "claude_cowork",
      "created_at": "2026-09-18T10:00:00Z",
      "updated_at": "2026-09-18T10:05:00Z"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

### Get remote session transcript

```
GET /v1/compliance/sessions/{session_id}
```

**Response:** Full session object with `transcript` array of turn objects.

---

## Local sessions

Local sessions are sessions that run on users' machines — Claude Code, and (as of Sep 18, 2026) Claude in Chrome.

> **Beta:** Claude in Chrome (`product_surface: "claude_in_chrome"`) is in beta as of Sep 18, 2026. Coverage of Claude Science and Claude for Microsoft 365 sessions is also in beta. Cowork and Claude Code are stable.

### List local sessions

```
GET /v1/compliance/local-sessions
```

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `product_surface` | string | Filter: `claude_code`, `claude_in_chrome` |
| `after` | string | Pagination cursor |
| `before` | string | Pagination cursor |
| `limit` | integer | Max results per page |

> **Note:** The local sessions endpoint does not support user-level filtering (no `user_id` parameter).

**Response:**

```json
{
  "sessions": [
    {
      "id": "local_sess_abc",
      "product_surface": "claude_in_chrome",
      "created_at": "2026-09-18T09:30:00Z",
      "updated_at": "2026-09-18T09:45:00Z"
    }
  ],
  "has_more": false
}
```

### Get local session transcript

```
GET /v1/compliance/local-sessions/{session_id}
```

---

## Transcript structure

Each session transcript is an array of turns. Turn objects include:

| Field | Description |
|-------|-------------|
| `role` | `"user"` or `"assistant"` |
| `content` | Array of content blocks (text, tool_use, tool_result) |
| `created_at` | ISO 8601 timestamp |

```json
{
  "id": "sess_abc123",
  "product_surface": "claude_in_chrome",
  "transcript": [
    {
      "role": "user",
      "content": [{"type": "text", "text": "Summarize this page"}],
      "created_at": "2026-09-18T09:30:01Z"
    },
    {
      "role": "assistant",
      "content": [{"type": "text", "text": "This page is about..."}],
      "created_at": "2026-09-18T09:30:03Z"
    }
  ]
}
```

---

## Claude in Chrome sessions (Sep 18, 2026)

Claude in Chrome sessions appear as **local sessions** (not remote) with `product_surface: "claude_in_chrome"`. Retrieve them via the `/v1/compliance/local-sessions` endpoint.

- Uses the same Compliance Access Key and `read:compliance_user_data` scope — no new credentials needed.
- No new client update required on the user's browser.
- Currently in beta for Claude Enterprise organizations.

---

## Example: fetch all Chrome sessions

```python
import requests

BASE_URL = "https://api.anthropic.com"
COMPLIANCE_KEY = "your-compliance-access-key"

headers = {
    "Authorization": f"Bearer {COMPLIANCE_KEY}",
    "anthropic-version": "2023-06-01",
}

def list_chrome_sessions():
    sessions = []
    cursor = None
    while True:
        params = {"product_surface": "claude_in_chrome", "limit": 100}
        if cursor:
            params["after"] = cursor
        resp = requests.get(f"{BASE_URL}/v1/compliance/local-sessions", headers=headers, params=params)
        resp.raise_for_status()
        data = resp.json()
        sessions.extend(data["sessions"])
        if not data.get("has_more"):
            break
        cursor = data.get("next_cursor")
    return sessions

def get_session_transcript(session_id: str) -> dict:
    resp = requests.get(f"{BASE_URL}/v1/compliance/local-sessions/{session_id}", headers=headers)
    resp.raise_for_status()
    return resp.json()
```

---

## Related

- [Manage Claude — Compliance API setup](https://platform.claude.com/docs/en/manage-claude/compliance-api-access)
- [Compliance content data](https://platform.claude.com/docs/en/manage-claude/compliance-content-data)
- [Managed Agents](./managed-agents.md)
