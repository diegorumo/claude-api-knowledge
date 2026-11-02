# Usage and Cost Analytics API (beta)

> **Last updated:** 2026-11-02  
> **Sources:** platform.claude.com/docs/en/manage-claude/usage-cost-analytics-api, platform.claude.com/docs/en/manage-claude/usage-cost-api, platform.claude.com/docs/en/manage-claude/analytics-api, platform.claude.com/docs/en/release-notes/overview (Oct 29, 2026)  
> **Status:** Beta since Oct 29, 2026. Beta header: `analytics-usage-cost-2026-08-24`

**Summary:** Two Admin API endpoints that report historical token usage and cost for a Claude Console or Claude Enterprise organization, filtered and grouped by model, workspace, API key, actor (user / service account / federated identity / API key), service tier and more. The same endpoints, parameters and response fields serve both kinds of organization.

| Endpoint                                  | Reports                        |
| ----------------------------------------- | ------------------------------ |
| `GET /v1/organizations/analytics/usage`   | Token counts and request counts |
| `GET /v1/organizations/analytics/cost`    | Cost in USD                    |

Send `anthropic-beta: analytics-usage-cost-2026-08-24` and an `anthropic-version` header on every request. The SDKs and the `ant` CLI send both for you and auto-paginate, so `limit` only sets the page size.

## Legacy APIs (still supported)

As of Oct 29, 2026 these are **legacy** but remain supported:

- The Usage and Cost API: `/v1/organizations/usage_report/messages` and `/v1/organizations/cost_report` (Claude Console organizations).
- The Claude Enterprise Analytics API report endpoints `usage_report`, `cost_report`, `user_usage_report` and `user_cost_report` under `/v1/organizations/analytics/`.

For new integrations the official docs recommend the Usage and Cost Analytics API. Migration notes are below.

## Authentication

- **Claude Console organizations:** an Admin API key, or another Admin API credential such as an OAuth token with the `org:admin` scope (sent as `authorization: Bearer`).
- **Claude Enterprise organizations:** a key with the `read:analytics` scope, created in claude.ai by the primary owner after the Analytics API is enabled for the organization.
- **Claude Platform on AWS:** both endpoints are available, through the Claude Platform on AWS base URL and authentication; AWS IAM authorizes them through the `ListAccountUsage` and `ListAccountCost` actions.
- **Claude Models on Google Cloud (Anthropic managed):** both endpoints are available through the organization-level base URL with Google Cloud credentials. `/usage` needs the `usageAnalytics.list` IAM permission.

The Admin API isn't available to individual accounts.

## Quick start

Daily token usage for one week, one row per model and day:

```python
import anthropic

client = anthropic.Anthropic()

rows = client.beta.organization.analytics.usage.list(
    starting_at="2026-09-01T00:00:00Z",
    ending_at="2026-09-08T00:00:00Z",
    time_bucket_width="1d",
    group_by=["model"],
    limit=10,
)

for row in rows:
    print(f"{row.starting_at} {row.group.model}: {row.usage.total_tokens}")
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const rows = await client.beta.organization.analytics.usage.list({
  starting_at: "2026-09-01T00:00:00Z",
  ending_at: "2026-09-08T00:00:00Z",
  time_bucket_width: "1d",
  group_by: ["model"],
  limit: 10,
});

for await (const row of rows) {
  console.log(`${row.starting_at} ${row.group.model}: ${row.usage.total_tokens}`);
}
```

```bash
curl --globoff "https://api.anthropic.com/v1/organizations/analytics/usage?\
starting_at=2026-09-01T00:00:00Z&\
ending_at=2026-09-08T00:00:00Z&\
time_bucket_width=1d&\
group_by[]=model&\
limit=10" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
  -H "x-api-key: $ANTHROPIC_API_KEY"
```

Response shape (one row shown; `group` lists every dimension, `null` unless grouped):

```json
{
  "data": [
    {
      "starting_at": "2026-09-03T00:00:00Z",
      "ending_at": "2026-09-04T00:00:00Z",
      "group": { "model": "claude-sonnet-5", "workspace_id": null, "actor": null, "...": null },
      "usage": {
        "uncached_input_tokens": 1284500,
        "cache_creation": { "ephemeral_1h_input_tokens": 0, "ephemeral_5m_input_tokens": 215300 },
        "cache_read_input_tokens": 3200000,
        "output_tokens": 891000,
        "total_tokens": 5590800,
        "server_tool_use": { "web_search_requests": 12 },
        "requests": 4210
      }
    }
  ],
  "has_more": true,
  "next_page": "page_...",
  "data_refreshed_at": "2026-09-18T16:02:11Z"
}
```

## How results are shaped

- **Rows:** each row has `starting_at` / `ending_at`, a `group` object holding every dimension, and `usage` (or `cost`) metrics.
- **Grouping:** a dimension is `null` unless listed in `group_by[]` (up to five dimensions). It is also `null` on rows with no value for it, such as `model` on code execution usage.
- **One row per combination** that has data. No `group_by[]` and no `time_bucket_width` gives a single total row for the range.
- **Time buckets (optional):** `time_bucket_width` = `1m`, `1h` or `1d` (UTC). Empty buckets produce no rows; fill them with zeros yourself.
- **Ordering:** rows are ranked by the `order_by` metric across the whole range, largest first (`order=asc` reverses). For a time series, set `time_bucket_width` and `order_by=time` (newest bucket first by default).
- **Filters:** most dimensions have a list filter in bracket notation, such as `models[]=claude-opus-5-5`, up to 100 values. Values of the same filter are ORed; different filters are ANDed. An unknown ID or misspelled model name matches nothing instead of returning an error.
- Some dimensions apply only to Claude Console organizations (such as `api_key_id`) or only to Claude Enterprise (such as `slack_channel_id`). A dimension that doesn't apply is `null` everywhere; `rbac_group_id` returns an error when teams aren't available.
- `limit`: rows per page, 1 to 1,000, default 20.

### Actors

`actor` is who a request is attributed to: a user, a service account, a federated cloud identity (AWS or Google Cloud principal), or the API key itself when none of those applies. Grouped `actor` values are objects with a `type`. Unattributed usage has `actor: null`. Personal and service account keys act as their linked identity (key in `api_key_id`); legacy workspace keys appear as both `actor` and `api_key_id`. `api_key_ids[]=apikey_...` matches every request the key authenticated; `actor_ids[]=apikey_...` matches only requests attributed to the key itself.

## Usage endpoint

`GET /v1/organizations/analytics/usage`. `order_by`: `total_tokens` (default), `output_tokens`, `uncached_input_tokens`, `requests` or `time`.

Top 25 actors by request count for a month:

```python
rows = client.beta.organization.analytics.usage.list(
    starting_at="2026-09-01T00:00:00Z",
    ending_at="2026-10-01T00:00:00Z",
    group_by=["actor"],
    order_by="requests",
    limit=25,
)
for row in rows:
    print(f"{row.group.actor}: {row.usage.requests}")
```

```typescript
const rows = await client.beta.organization.analytics.usage.list({
  starting_at: "2026-09-01T00:00:00Z",
  ending_at: "2026-10-01T00:00:00Z",
  group_by: ["actor"],
  order_by: "requests",
  limit: 25,
});
for await (const row of rows) {
  console.log(`${JSON.stringify(row.group.actor)}: ${row.usage.requests}`);
}
```

## Cost endpoint

`GET /v1/organizations/analytics/cost` takes the same filters, dimensions and time buckets. Each row's `cost` has:

- `amount`: cost after discounts, before credits
- `list_amount`: cost at list prices
- `currency`: always `"USD"`
- `requests`

Amounts are **decimal strings in cents** with six decimal places: `"41280.000000"` is $412.80. `order_by`: `amount` (default), `list_amount` or `time`.

Two extra dimensions exist only here and don't count toward the five-dimension limit (and `limit` doesn't apply to them): `group_by[]=cost_type` and `group_by[]=token_type`. With either, `requests` is `null` on every row.

```python
rows = client.beta.organization.analytics.cost.list(
    starting_at="2026-09-01T00:00:00Z",
    ending_at="2026-10-01T00:00:00Z",
    group_by=["model", "cost_type", "token_type"],
)
for row in rows:
    print(f"{row.group.model} {row.group.cost_type} {row.group.token_type}: {row.cost.amount}")
```

```typescript
const rows = await client.beta.organization.analytics.cost.list({
  starting_at: "2026-09-01T00:00:00Z",
  ending_at: "2026-10-01T00:00:00Z",
  group_by: ["model", "cost_type", "token_type"],
});
for await (const row of rows) {
  console.log(`${row.group.model} ${row.group.cost_type} ${row.group.token_type}: ${row.cost.amount}`);
}
```

> **Priority Tier:** requests under a Priority Tier commitment show on `/usage` with `service_tier: "priority"` but add zero token cost to `amount` / `list_amount`; the commitment fee is billed separately. Web search on those requests is billed at the standard rate.

## Date ranges and data freshness

- RFC 3339 timestamps with UTC offset. `starting_at` inclusive, `ending_at` exclusive.
- `starting_at` must be on or after **Jan 1, 2026** and within the last **365 days**. A request spans at most **31 days**; `ending_at` defaults to 31 days after `starting_at` or now, whichever is earlier.
- `ending_at` is required with `time_bucket_width`; with `1m` buckets a request spans at most **24 hours**.
- Both ends round down to a bucket boundary (or to the hour with no bucket).
- Every response has `data_refreshed_at`; usage after it is missing or partial. Data typically refreshes within **4 hours** and can take up to **24 hours**. Values can be revised until about 7 days after the end of the calendar month. All pages of one query read the same snapshot.

## Migrating from the legacy Usage and Cost API (Claude Console)

| Usage and Cost API (legacy)                                   | Usage and Cost Analytics API                                                                                       |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| No beta header                                                | `anthropic-beta: analytics-usage-cost-2026-08-24`                                                                  |
| Data typically within 5 minutes                               | Typically within 4 hours, up to 24 hours                                                                           |
| `data` = time buckets (oldest first, empty ones included)     | `data` = ranked rows, each with its own `starting_at` / `ending_at`, `group`, and `usage` or `cost`                |
| `bucket_width`, default `1d`                                  | `time_bucket_width`, no default (omit for whole-range totals)                                                      |
| `limit` counts time buckets                                   | `limit` counts rows (1–1,000, default 20)                                                                          |
| Any date range                                                | On or after Jan 1, 2026, last 365 days, max 31 days per request, max 24 hours with `1m`                            |
| `account_ids[]`, `service_account_ids[]`, `group_by[]=account_id` / `service_account_id` | `actor_ids[]` and `group_by[]=actor` or `actor_type`                                    |
| `context_window[]`                                            | `context_windows[]`                                                                                                |
| `group_by[]=description` on cost                              | Removed; group by `model`, `cost_type`, `token_type`, `context_window`, `service_tier`, `inference_geo`, `speed`  |
| Unknown API key or workspace ID returns an error              | Matches nothing                                                                                                    |
| `workspace_id: null` for the default workspace                | The default workspace's real `wrkspc_` ID                                                                          |
| `inference_geo: "not_available"`, `service_tier: "standard"` when absent | `null` for both                                                                                         |
| Cost `amount` like `"123.45"`; tier `standard` or `batch`      | Six decimals (`"123.450000"`), plus `list_amount`; the actual tier                                                 |
| Code execution not in usage; no request counts                | Code execution rows with `model: null`; `requests` on both endpoints                                               |

To reproduce the legacy daily report: `time_bucket_width=1d`, same `group_by[]`, `limit=1000`, `order_by=time&order=asc`, then group rows by `starting_at` and fill missing days with zeros.

## Migrating from the Claude Enterprise usage and cost reports

The legacy reports return at most the top 100 groups per bucket. The new endpoints return every combination, can group by `actor` / `actor_type` alongside other dimensions on both usage and cost, and serve Console organizations too. Key, earliest date, 365-day lookback, 31-day span, most filter and dimension names, metric fields, `data_refreshed_at` and the organization-level rate limit carry over.

| Claude Enterprise reports (legacy)                          | Usage and Cost Analytics API                                                    |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Time buckets, top 100 groups each; `limit` counts buckets   | Every combination as ranked rows; `limit` counts rows                           |
| `bucket_width`                                              | `time_bucket_width` (same values)                                               |
| `ending_at` optional; `1m` buckets up to 31 days            | `ending_at` required with buckets; `1m` up to 24 hours                          |
| `user_usage_report` / `user_cost_report`                    | `group_by[]=actor&actor_types[]=user_actor`                                     |
| `product: "claude-tag"`                                     | `product: "claude_tag"` (`products[]` accepts either)                           |
| Unknown `models[]` name returns 400                         | Matches nothing                                                                 |
| `user_ids[]`                                                | `actor_ids[]` (same `user_...` IDs)                                             |
| `exclude_deleted_users`                                     | Requires `group_by[]=actor` (400 otherwise)                                     |

Per-user cost for a month, broken down by product:

```python
rows = client.beta.organization.analytics.cost.list(
    starting_at="2026-09-01T00:00:00Z",
    ending_at="2026-10-01T00:00:00Z",
    actor_types=["user_actor"],
    group_by=["actor", "product"],
    limit=100,
)
for row in rows:
    actor = row.group.actor
    if actor is not None and actor.type == "user_actor":
        print(f"{actor.user_id} {row.group.product}: {row.cost.amount}")
```

```typescript
const rows = await client.beta.organization.analytics.cost.list({
  starting_at: "2026-09-01T00:00:00Z",
  ending_at: "2026-10-01T00:00:00Z",
  actor_types: ["user_actor"],
  group_by: ["actor", "product"],
  limit: 100,
});
for await (const row of rows) {
  const { actor } = row.group;
  if (actor?.type === "user_actor") {
    console.log(`${actor.user_id} ${row.group.product}: ${row.cost.amount}`);
  }
}
```

Per-user Claude Code cost: `products[]=claude_code&group_by[]=actor` on the cost endpoint.

## Related

- [Authentication](./authentication.md)
- [Compliance API](./compliance-api.md)
- [Usage and Cost Analytics API (official)](https://platform.claude.com/docs/en/manage-claude/usage-cost-analytics-api)
- [Usage and Cost API (official, legacy)](https://platform.claude.com/docs/en/manage-claude/usage-cost-api)
