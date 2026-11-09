# Organization Security: IP Allowlisting and Member Sign-Out (beta)

> **Last updated:** 2026-11-09  
> **Sources:** platform.claude.com/docs/en/manage-claude/ip-allowlist, platform.claude.com/docs/en/manage-claude/user-management (Sign a member out everywhere; Respond to a lost or stolen device), platform.claude.com/docs/en/release-notes/overview (Nov 5, 2026)  
> **Status:** Both in beta since Nov 5, 2026

## IP allowlisting (beta)

An organization admin or owner sets a list of IP ranges for the whole organization. For each covered request, Anthropic compares the request's public IP with the list and denies it if it isn't allowed. This limits where a leaked API key (or, in Claude Enterprise, a stolen sign-in) can be used. The docs recommend combining it with short [key expiration](https://platform.claude.com/docs/en/manage-claude/authentication#key-expiration) or Workload Identity Federation.

- **API requests** from outside the list are rejected. The Messages API returns **403 `permission_error`** with the message `Your IP address isn't in this organization's allowed range.`
- **People** connecting from a disallowed address see a page titled **Your current network can't access this organization**. Admins are blocked too, unless you give them a way back in (see below).

### What it covers

| Organization      | Covered                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Claude Platform   | Claude Console UI; Claude API endpoints Messages, Message Batches, Token Counting, Models, Files, Skills (API key or WIF token); Managed Agents endpoints Agents, Sessions, Environments, Vaults, Memory stores; Admin API and Compliance API (with a key that covers only this organization)                                                                              |
| Claude Enterprise | Claude on web, desktop and mobile; Claude Code when signed in with a Claude account; Claude in Chrome, Office add-ins, Cowork, Remote Tools (including connectors and artifacts, except artifacts shared outside the org); Admin API and Compliance API. Covers owners too                                                                                                 |

**Not covered:**

- Claude Platform: the Managed Agents runtime's own networking (only requests to the endpoints are covered); Amazon Bedrock, Claude Platform on AWS, Claude Models on Google Cloud, Microsoft Foundry.
- Claude Enterprise: Claude Design, Claude in Slack, Claude Academy. A task already running in a chat can continue for up to 24 hours after the user is blocked.

In Claude Enterprise, Claude Code users outside the list get a 403 API error with the same message. Features that connect through `bridge.claudeusercontent.com` (Claude in Chrome, Cowork, Remote Tools, work across apps) need that host's egress address listed if your proxy routes it differently from `claude.ai` and `api.anthropic.com`.

### Setting it up

1. Open **Identity and access** in the Claude Console (Claude Platform) or **claude.ai > Organization settings > Organization and access** (Claude Enterprise; owners and primary owners only).
2. Turn on the **IP allowlist** toggle, add addresses or ranges, keep the **Allow admins (owners) to access this settings page from any IP address** checkbox selected (recommended), and save.
3. Changes can take **up to 15 minutes** to apply. Watch for 403 `permission_error` responses, including weekly or daily jobs.

**Entry format:** IPv4 or IPv6, single address or CIDR range, no broader than `/8` (IPv4) or `/32` (IPv6). Up to **1,000 entries** per organization. Public egress addresses only (not private ranges such as `10.0.0.0/8`).

The settings page checks only your current address on save. List every network first (data centers, CI, offices, VPNs, proxies, self-hosted sandbox workers). Allowlist changes are recorded in the Compliance API Activity Feed. If everyone is locked out, Anthropic support can turn the allowlist off.

## Sign a member out everywhere (beta, Claude Enterprise)

`POST /v1/organizations/users/{user_id}/sign_out` signs a member out of Claude on every browser and device without removing them from the organization. Use it for a lost or stolen device, a security incident, or while you investigate an account.

| Item          | Value                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------- |
| Beta header   | `anthropic-beta: admin-user-sign-out-2026-10-08` (missing it returns 400; SDK and CLI send it)    |
| Scope         | `write:members` (add `read:members` to look members up by email)                                  |
| Key           | Admin API key created in claude.ai by that organization's primary owner; one key per organization |
| Body          | None. Pass the member `id` from List members                                                      |
| Response      | 200 with the unchanged member object; safe to retry                                               |

```python
import anthropic

client = anthropic.Anthropic()

user = client.beta.organization.users.sign_out("user_01AbCdEfGhIjKlMnOpQrSt")

print(f"id: {user.id}")
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const user = await client.beta.organization.users.signOut("user_01AbCdEfGhIjKlMnOpQrSt");

console.log(`id: ${user.id}`);
```

```bash
curl -X POST "https://api.anthropic.com/v1/organizations/users/user_01AbCdEfGhIjKlMnOpQrSt/sign_out" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2026-10-21" \
  -H "anthropic-beta: admin-user-sign-out-2026-10-08"
```

CLI: `ant beta:organization:users sign-out --user-id user_01AbCdEfGhIjKlMnOpQrSt`. Without a key, an owner can use **Sign out everywhere** on the Members page of claude.ai organization settings (not on their own account).

**What it ends:** all sign-in sessions for the person's account on every browser and device; trusted devices; OAuth tokens issued for this organization (almost always at once, within an hour at most). It also clears skip-2FA browser records and removes push-notification registrations (asynchronous; a 200 doesn't confirm these two). Sessions, trusted devices and push registrations belong to the account, so the sign-out also reaches the person's other organizations and individual plan. The member keeps their seat and role and can sign in again right away.

**What it doesn't end:** API keys (including Admin API keys the person created); OAuth tokens for other organizations or issued to the account rather than an organization; Claude Code cloud sessions and routines; computers linked to Cowork; a sign-in already in progress (can finish up to 5 minutes later, so re-send if logs show a later sign-in). To keep someone out, remove them or disable their account in your identity provider.

The primary owner can't be signed out (400 `cannot_sign_out_primary_owner`). Each completed sign-out is recorded as `org_user_trusted_devices_revoked` in the audit log export and the Activity Feed.

| Status                                  | Meaning                                                                                           |
| --------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 200                                     | Completed (also when nothing was left to end)                                                     |
| 400 `cannot_sign_out_primary_owner`     | Member is the primary owner                                                                       |
| 400 `organization_not_enrolled_in_beta` | Endpoint not available to your organization                                                       |
| 400                                     | Missing `anthropic-beta` value, or the path `id` doesn't start with `user_`                       |
| 401                                     | Key missing, invalid, deleted, or created for a parent org and not bound to one organization      |
| 403                                     | Key lacks `write:members`                                                                         |
| 404                                     | No member with that ID in your organization                                                       |
| 429                                     | Over the shared Admin API rate limit (see `retry-after`)                                          |
| 503                                     | A needed service is temporarily unavailable; what was ended stays ended. Send again               |

## Related

- [Authentication](./authentication.md)
- [Compliance API](./compliance-api.md)
- [Inference Hooks](./inference-hooks.md)
