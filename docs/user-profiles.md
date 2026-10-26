# User Profiles and Organization Webhooks

> **Last updated:** 2026-10-26  
> **Sources:** platform.claude.com/docs/en/manage-claude/user-profiles, platform.claude.com/docs/en/manage-claude/organization-webhooks, platform.claude.com/docs/en/release-notes/overview (Oct 20 and Oct 21, 2026)

**Summary:** User profiles tell Anthropic which of your end customers each API request is for, so a safety enforcement action can apply to one end customer instead of your whole organization. Launched on Oct 20, 2026 on the non-beta `/v1/user_profiles` endpoints (`client.user_profiles` in Python, `client.userProfiles` in TypeScript); earlier SDK releases had them in beta behind `user-profiles-*` headers. Profiles aren't billable. Since Oct 21, 2026 a profile also shows whether Anthropic has restricted or banned that end customer, and organization webhooks (beta) can notify you when that changes.

## Who gets a profile

| Who the traffic is for                                    | Profile                                            | `access_type`                                 |
| --------------------------------------------------------- | -------------------------------------------------- | --------------------------------------------- |
| A signed-up user of your product                          | One per user, created at sign-up                   | `application` (default)                       |
| A company you resell API access to (for example a gateway) | One per company, not per person in it              | `passthrough` (must be sent)                  |
| Your own organization                                     | None; a request with no profile ID is your own usage | Not applicable                              |

Profiles belong to an organization, not a workspace. Keep end-customer traffic in a separate organization from your own internal usage. You manage profiles on the Claude API, Claude Platform on AWS, or a Microsoft Foundry deployment hosted on Anthropic. Amazon Bedrock and Claude Models on Google Cloud (Google managed) have no `/v1/user_profiles` endpoints.

## Create a profile

`POST /v1/user_profiles` is not idempotent: each call creates a new profile, even with the same `reference_id`. Create it once, during onboarding, and store the `id`. Every field is optional; leave unknown values out rather than sending placeholders (values in `external_user_details` can be replaced but not cleared).

```python
import anthropic

client = anthropic.Anthropic()

profile = client.user_profiles.create(
    name="Acme Analytics, Inc.",
    access_type="passthrough",
    external_user_details={"reference_id": "cust-8f2a1c", "country": "US"},
)
print(profile.id)
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const profile = await client.userProfiles.create({
  name: "Acme Analytics, Inc.",
  access_type: "passthrough",
  external_user_details: { reference_id: "cust-8f2a1c", country: "US" }
});
console.log(profile.id);
```

`external_user_details` accepts `entity_type`, `country`, `reference_id`, `onboarded_at`, `account_status`, `name_hash` and `email_hash`. There is no email field; send a hash you compute yourself, normalized the same way for every profile.

## Attribute each request

Send the profile ID in the `anthropic-user-profile-id` header. The SDKs take `user_profile_id`:

```python
message = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
    user_profile_id=profile_id,
)
```

```typescript
const message = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  user_profile_id: profileId
});
```

- Works the same way on token counting, Message Batches (applies to every request in the batch) and OpenAI SDK compatibility requests.
- The Claude API, Claude Platform on AWS and Foundry (hosted on Anthropic) return `400 invalid_request_error` for a malformed ID or one from another organization. A batch is still created with a foreign but well-formed ID; each request then errors.
- **Claude in Amazon Bedrock:** put `user_profile_id` at the top level of the request body. **Claude Models on Google Cloud (Google managed):** send the header. Create the profiles on the Claude API or Claude Platform on AWS, and link each AWS account / Google Cloud project in the Verification Portal (`portal.anthropic.com`, **Linked accounts**). These platforms check only the ID's format. Not supported on the older Bedrock `InvokeModel` / `Converse` integration.
- For a `passthrough` profile, also send a per-person identifier in the request's `metadata.user_id` when one person makes the request.

## Enforcement status (Oct 21, 2026)

Every response that returns a profile includes `enforcement_status` and `enforcement_actions`. Send these requests **without** a `user-profiles-` value in `anthropic-beta`: `user-profiles-2026-09-04` and earlier values leave the fields out (and make the `enforcement_status[]` filter return `400`).

```json
{
  "type": "user_profile",
  "id": "uprof_011CZkZCu8hGbp5mYRQgUmz9",
  "enforcement_status": "restricted",
  "enforcement_actions": [
    {
      "type": "restriction",
      "id": "enac_011CZkZEq4TnWb7xLfRgPm2v",
      "reason": "fraud_or_abuse",
      "action": null,
      "opened_at": "2026-10-02T19:30:00Z",
      "expires_at": null
    }
  ]
}
```

- `enforcement_status`: `unrestricted`, `restricted` (an action is open, none is a ban) or `banned`. Treat unknown values as `restricted`.
- Each action: `type` (`restriction` or `ban`), `id` (`enac_` prefix), `reason` (`unsupported_region`, `fraud_or_abuse`, `high_harm`, `usage_policy_violation`, `account_compromise` or `unspecified`), `action` (what the end customer can do, or `null`), `opened_at`, `expires_at` (or `null`).
- While an action is open, Anthropic refuses some or all inference requests that carry the profile ID.
- Find affected profiles with `GET /v1/user_profiles?enforcement_status[]=restricted&enforcement_status[]=banned`. A change doesn't reliably move `updated_at`.
- Both fields are set by Anthropic; sending either on create or update returns `400`.
- On Claude Platform on AWS the fields don't report enforcement: always `unrestricted` and an empty list.

## Update and delete

`POST /v1/user_profiles/{id}` changes only the fields you send. `name` and `access_type` replace; each `external_user_details` field replaces (`null` is rejected); `metadata` merges key by key, and an empty string deletes a key.

```python
profile = client.user_profiles.update(
    profile_id,
    external_user_details={"account_status": "suspended"},
    metadata={"trial_cohort": ""},
)
```

## Organization webhooks (beta, Oct 20, 2026)

Admin API endpoints that register HTTPS endpoints for enforcement events. Beta header `webhooks-2026-10-07` (the SDK and CLI send it). Access by request through your Anthropic account team; not available with HIPAA readiness enabled. Credentials: an Admin API key, an OAuth bearer token with `org:admin`, or a personal / service account key not scoped to a workspace.

| Event                                      | Trigger                                                      |
| ------------------------------------------ | ------------------------------------------------------------ |
| `user_profile.enforcement_action_opened`   | Anthropic opens an enforcement action on a user profile      |
| `user_profile.enforcement_action_resolved` | An action ends (resolved, or its `expires_at` passes)        |

Events carry IDs only (`data.id` is the profile, `data.enforcement_action_id` the action); retrieve the profile to see what changed. Deleting a profile ends its actions without a resolved event. Actions opened before you registered an endpoint send no event, so also list profiles by `enforcement_status[]`.

| Task                    | Request                                                                    |
| ----------------------- | -------------------------------------------------------------------------- |
| List endpoints          | `GET /v1/organizations/webhooks`                                           |
| Create an endpoint      | `POST /v1/organizations/webhooks`                                          |
| Get one endpoint        | `GET /v1/organizations/webhooks/{webhook_endpoint_id}`                     |
| Update an endpoint      | `POST /v1/organizations/webhooks/{webhook_endpoint_id}`                    |
| Delete an endpoint      | `DELETE /v1/organizations/webhooks/{webhook_endpoint_id}`                  |
| Replace signing secret  | `POST /v1/organizations/webhooks/{webhook_endpoint_id}/regenerate_signing_secret` |

```python
endpoint = client.beta.organization.webhooks.create(
    url="https://example.com/anthropic-webhooks",
    name="Enforcement actions",
    enabled_events=[
        "user_profile.enforcement_action_opened",
        "user_profile.enforcement_action_resolved",
    ],
)
print(endpoint.id)
```

```typescript
const endpoint = await client.beta.organization.webhooks.create({
  url: "https://example.com/anthropic-webhooks",
  name: "Enforcement actions",
  enabled_events: [
    "user_profile.enforcement_action_opened",
    "user_profile.enforcement_action_resolved"
  ]
});
console.log(endpoint.id);
```

- Endpoint URL: HTTPS on port 443, public domain name (no IP address); two enabled endpoints can't share a URL. Up to 32 endpoints per organization (disabled ones count).
- `signing_secret` (`whsec_` prefix) is returned only on create and on secret regeneration. Regenerating invalidates the old secret at once.
- Deliveries follow [Standard Webhooks](https://www.standardwebhooks.com/) (`webhook-id`, `webhook-timestamp`, `webhook-signature`). Verify the signature, return `2xx` within 5 seconds, and deduplicate on the event `id`. Failed deliveries are retried a few times over a few minutes, then dropped.
- Anthropic disables an endpoint that returns a `3xx`, resolves to a non-public IP, or keeps failing; send `"state": "enabled"` to turn it back on. In an update, `enabled_events` replaces the whole list (an empty list keeps the old one).
- Lists return 20 endpoints by default (`limit` up to 100); page with `after_id` = `last_id` while `has_more` is `true`.

## Related

- [Messages API](./messages-api.md)
- [Token Counting](./token-counting.md)
- [User profiles (official)](https://platform.claude.com/docs/en/manage-claude/user-profiles)
- [Organization webhooks (official)](https://platform.claude.com/docs/en/manage-claude/organization-webhooks)
