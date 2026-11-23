# Linked Billing (Claude Platform OAuth)

> **Last updated:** 2026-11-23  
> **Sources:** platform.claude.com/docs/en/manage-claude/oauth-apps/organization-billing-connections, platform.claude.com/docs/en/release-notes/overview (Nov 17, 2026)

**Status:** Beta (launched Nov 17, 2026). Beta header `user-profiles-billing-2026-10-01`. Claude Platform OAuth is a Beta Service: endpoints, parameters and behavior can change during the beta.

**Summary:** Your customers can pay for the Claude usage in your app through their own Claude Platform organization. A customer links their Claude organization to your app once (the API calls the link a *connection*). After that, you choose on each `/v1/messages` or `/v1/chat/completions` request whether the customer or you pays, with the `billing_policy` field. In the Claude Console, customers can set a spend cap for your app or unlink it. If a request you mark for the customer can't be billed to them, the API refuses it rather than billing you.

## Eligibility and supported endpoints

- **Eligible customers:** organizations that pay Anthropic directly for API usage (prepaid credits or invoiced billing). Organizations billed through a cloud marketplace (AWS, Google Cloud, Azure) and organizations with only a Claude subscription plan can't be billed this way; requests for them get 403 `user_profile_not_billable`.
- Requests run **in your organization**: your rate limits, model access, service tier, data retention and encryption keys apply. The customer's own ZDR, BAA and CMEK arrangements don't apply. Don't link customers that need HIPAA, FERPA or CMEK coverage for this usage.
- **`/v1/messages`:** supported. Known limitation: a request with `bill_to: "user_profile"` can't use billable code execution features (400); send those with `bill_to: "self"` or without `billing_policy`.
- **`/v1/chat/completions`:** supported.
- **`/v1/messages/count_tokens`:** accepts `billing_policy` and ignores it.
- **Message Batches:** not supported for billing the customer. A batch request with `bill_to: "user_profile"` is `errored` (`invalid_request_error`) and nobody is billed for it; the batch's other requests run.
- Some credits can't pay for these requests, including the monthly API credits that come with Claude Max, Ultra and Team plans, and credits from Anthropic's partner, VC and startup programs.

## One-time setup (Admin role)

1. **Create the OAuth app** in the Console ([OAuth apps](https://platform.claude.com/settings/oauth-apps) → **Create**), with your redirect URI, and select **Bill and link to organizations**. Copy the public client ID. A new app starts in testing (**Who can link: Test users only**).
2. **Set up client authentication.** There is no client secret: your back end sends a client assertion JWT from your identity provider (a `private_key_jwt` variation, RFC 7523) to the PAR, token and revocation endpoints on `auth.claude.com`, configured through [Workload Identity Federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation).
3. **Set up your back end's identity:** a service account plus a second federation rule. Select that service account as the app's **operator service account**; only it can send inference requests that bill linked user profiles. A service account can be the operator of only one OAuth app.

Each customer's Claude organization maps to exactly one of your [user profiles](./user-profiles.md), and each user profile to one Claude organization (per app).

## Linking each customer (Steps 4–8)

4. Create a user profile for the customer's Claude organization and store its ID.
5. Create a pushed authorization request (`POST https://auth.claude.com/v1/oauth/par`, scope `billing:attribution`) with a `state` value and a PKCE code challenge; the response gives a `request_uri`.
6. Redirect the user to `https://auth.claude.com/oauth/authorize` with only `client_id` and `request_uri`. The person who approves must have the Admin or Billing role in the customer's organization.
7. The user selects an organization, reads how billing works, optionally sets a monthly spend limit for your app, and clicks **Link and authorize**.
8. On the callback, check `state` and exchange the code (single use, expires after 5 minutes) at `POST https://auth.claude.com/v1/oauth/token` with the PKCE verifier and your client assertion. Store the returned connection ID. A new connection can take about 10 seconds to start working.

## Billing a request

Send the `anthropic-user-profile-id` header on every call for a linked customer. Calls that set `billing_policy` must also send the `user-profiles-billing-2026-10-01` beta header.

### Your customer pays

```python
BILLING_BETA = "user-profiles-billing-2026-10-01"

def message_billed_to_user(user_profile_id: str, prompt: str) -> BetaMessage:
    return client.beta.messages.create(
        model="claude-opus-5-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
        betas=[BILLING_BETA],
        extra_headers={"anthropic-user-profile-id": user_profile_id},
        extra_body={
            "billing_policy": {"type": "billing_policy", "bill_to": "user_profile"}
        },
    )
```

```typescript
const client = new Anthropic();

const message = await client.beta.messages.create({
  model: "claude-sonnet-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  betas: ["user-profiles-billing-2026-10-01"],
  user_profile_id: "uprof_01HV9Z",
  billing_policy: { type: "billing_policy", bill_to: "user_profile" }
});
```

By default the operator service account's own credential can bill **every** customer linked through the app; if the header names the wrong linked profile, that profile's organization is billed and no error tells you. A per-connection OAuth access token can bill only its own customer. Only set `bill_to: "user_profile"` on usage the customer directs you to make, never for your own testing, evals or other customers.

### You pay

Omit `billing_policy` (the billing beta header is then optional), or set `bill_to: "self"`:

```python
def message_billed_to_you(user_profile_id: str, prompt: str) -> Message:
    return client.messages.create(
        model="claude-opus-5-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}],
        user_profile_id=user_profile_id,
    )
```

```typescript
const message = await client.messages.create({
  model: "claude-sonnet-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }],
  user_profile_id: "uprof_01HV9Z"
});
```

A request authenticated with a connection's access token can't be billed to you: it is billed to that customer even without `billing_policy`, and is refused if it sets `bill_to: "self"` or names another profile.

## Expected errors (`bill_to: "user_profile"`)

On `/v1/messages` the code is in `error.details.error_code`; `/v1/chat/completions` returns the same status and message in OpenAI format, without the code. Anthropic never silently bills you instead.

| Status | Code | Meaning |
| ------ | ---- | ------- |
| 403 `permission_error` | `user_profile_not_billable` | Can't bill this customer (wrong credential or profile, no active connection, ineligible organization). Not retryable, except in the first ~10 seconds after a new connection. |
| 402 `billing_error` | `linked_organization_billing_unavailable` | The customer's organization can't be billed right now (e.g. suspended, or credit ran out mid-reply). Not retryable. |
| 402 `billing_error` | `linked_organization_spend_limit_reached` | The customer's spend cap for your app or organization monthly spend limit is reached, or no credit can pay. Not retryable. |
| 503 `overloaded_error` | — | Billing eligibility couldn't be confirmed. Retry with backoff, honor `retry-after`. |
| 400 `invalid_request_error` | `user_profile_required` | `bill_to: "user_profile"` without the `anthropic-user-profile-id` header. |
| 400 `invalid_request_error` | — | Malformed or unknown profile ID, or a connection token used with `bill_to: "self"` or another profile. |

For the 403 and 402 cases you can still serve the customer at your own cost by resending with `bill_to: "self"` (authenticated with the operator service account's own credential, not a connection token). A successful response doesn't say who was billed, so keep your own record.

## Managing connections

- **Link status:** `GET /v1/user_profiles/{id}?oauth_app_id=YOUR_CLIENT_ID` with the `user-profiles-search-2026-08-30` beta header, as the operator service account. `connection_status.type` is `connected`, `disconnected` (with `last_revoked_at` / `last_revoked_reason`) or `not_connected`.
- **Unlink:** in the Console (User profiles page), or `POST https://auth.claude.com/v1/oauth/revoke` with the `connection_id` (or `user_profile_id`) and your client assertion.
- Customers can unlink at any time on their [Linked platforms](https://platform.claude.com/linked-platforms) page. You get **no webhook and no email** when they do; requests then get 403 `user_profile_not_billable`.
- **Usage:** the Console User profiles page and the [Usage and Cost Analytics API](./usage-cost-analytics-api.md) (group by `user_profile_id`) count both kinds of request. On the cost endpoint, customer-paid requests add 0 to `amount` and their list price to `list_amount`.

## Spend caps

Customers set a monthly spend cap for your app in whole USD (on the approval page or on Linked platforms), plus an optional default for all linked apps. When reached, billed requests get a 402 until 00:00 UTC on the first day of the next month or until the cap changes. The cap counts token costs and web search fees at list prices. You can't read a customer's cap.

While the app is in testing, in review or not approved, a spend limit is required on the approval page and capped at **$50 USD**, and it can't be removed. After the app is live the limit is optional.

## Related

- [User profiles](./user-profiles.md)
- [Usage and Cost Analytics API](./usage-cost-analytics-api.md)
- [Linked billing (official)](https://platform.claude.com/docs/en/manage-claude/oauth-apps/organization-billing-connections)
