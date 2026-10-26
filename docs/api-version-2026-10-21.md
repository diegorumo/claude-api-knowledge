# API Version 2026-10-21

> **Last updated:** 2026-10-26  
> **Sources:** platform.claude.com/docs/en/api/migration-2026-10-21, platform.claude.com/docs/en/api/versioning, platform.claude.com/docs/en/cli-sdks-libraries/overview, platform.claude.com/docs/en/release-notes/overview (Oct 21, 2026)

**Summary:** API version `2026-10-21` (released Oct 21, 2026) is the first new `anthropic-version` since `2023-06-01`. It has breaking changes in the Messages API, the Admin API and the Claude Enterprise Analytics API. New major versions of every SDK and the `ant` CLI send it by default, and they have their own breaking changes and a new license (Apache 2.0). `2023-06-01` keeps working.

## Where it's available

The Claude API, Claude Platform on AWS, Claude Models on Google Cloud (Anthropic managed), Microsoft Foundry and Claude Models on Google Cloud (Google managed). On Amazon Bedrock, only for Claude 4.7 and later models: for older Bedrock models, stay on your current SDK version (or keep sending the API version you send today).

## Switching

Without an SDK, set the header:

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2026-10-21" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello, Claude"}]
  }'
```

With an SDK, each major version sends one API version. These are the first versions that send `2026-10-21`:

| Tool       | First version on `2026-10-21` | Package                                                                    |
| ---------- | ----------------------------- | -------------------------------------------------------------------------- |
| Python     | 2.0.0                         | `anthropic`                                                                |
| TypeScript | 1.0.0                         | `@anthropic-ai/sdk`                                                        |
| C#         | 13.0.0                        | `Anthropic`                                                                |
| Go         | 2.0.0                         | `github.com/anthropics/anthropic-sdk-go/v2` (new import path)              |
| Java       | 3.0.0                         | `com.anthropic:anthropic-java`                                             |
| PHP        | 1.0.0                         | `anthropic-ai/sdk` (run `composer require "anthropic-ai/sdk"`)             |
| Ruby       | 2.0.0                         | `anthropic`                                                                |
| `ant` CLI  | 2.0.0                         | `github.com/anthropics/anthropic-cli/v2` for `go install`                  |

- **TypeScript platform packages:** v1 of `@anthropic-ai/bedrock-sdk`, `@anthropic-ai/vertex-sdk`, `@anthropic-ai/foundry-sdk`, `@anthropic-ai/aws-sdk` and `@anthropic-ai/google-cloud-sdk` needs v1 of `@anthropic-ai/sdk`. Move them in the same upgrade, or your package manager installs a second, older copy of the SDK.
- **CLI via `go install`:** the old path (`github.com/anthropics/anthropic-cli/cmd/ant@latest`) keeps installing the latest v1 with no warning. Use `github.com/anthropics/anthropic-cli/v2/cmd/ant@latest`.
- Latest versions on the SDK overview page (checked 2026-10-26): Python 2.3.0, TypeScript 1.3.0, C# 13.3.0, Go 2.3.0, Java 3.3.0, PHP 1.3.0, Ruby 2.3.0, `ant` CLI 2.3.0.

Release the SDK upgrade together with the code fixes below, then test against `2026-10-21` before you ship.

## Breaking changes: Messages API

These apply to `POST /v1/messages`, token counting, and each request in a Message Batch created with `2026-10-21`.

| Change                                                    | What returns 400                                                                                       | What to do                                                                                                                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Document blocks in assistant messages                     | A `document` block placed directly in an `assistant` message                                           | Send documents in `user` messages. A document nested in another block (for example a web fetch tool result) is still accepted.                                   |
| Citations in user messages                                | A `text` block with a non-empty `citations` array in a `user` message, directly or in a `search_result` | Remove `citations` from those blocks. Citations on `assistant` text blocks and `"citations": {"enabled": true}` on documents / search results are still accepted. |
| `temperature`, `top_p`, `top_k` removed                   | Any of the three, on every model, even at the default value                                            | Remove them. Requests sample with the defaults (`temperature` 1.0).                                                                                              |

On models that only accept the defaults (Claude 4.7 and later) nothing changes. On models that accept other values, such as Claude Sonnet 4.6, a request that lowered `temperature` loses that effect, and nothing replaces it. If a request depends on a non-default value, keep it on `2023-06-01`.

## Breaking changes: Admin API and Claude Enterprise Analytics API

Responses no longer include the old names of renamed fields.

| API                  | Endpoints                                                            | Old name       | New name                                    |
| -------------------- | -------------------------------------------------------------------- | -------------- | ------------------------------------------- |
| Admin                | RBAC groups (create, list, retrieve, update)                         | `roles`        | `role_ids`                                  |
| Admin                | RBAC group members (list, add, and the remove response)              | `group_id`     | `rbac_group_id`                             |
| Admin                | RBAC roles (list, retrieve)                                          | `name`         | `display_name`                              |
| Admin                | Organization and workspace rate limits (list; set-workspace response) | `group_type`   | `group.type`                                |
| Admin                | API keys (list, retrieve, update)                                    | `workspace_id` | `scope` (different values, see below)       |
| Enterprise Analytics | Per-user usage and cost reports                                      | `actor.email`  | `actor.email_address`                       |
| Enterprise Analytics | Activity summaries                                                   | `summaries`    | `data`                                      |

The `group_type` query parameter on the rate-limit list endpoints and the `workspace_id` filter on the API key list endpoint are unchanged.

**`workspace_id` on API keys.** In `2023-06-01` it was the bound workspace's ID, or `null` for both the Default Workspace and an all-workspaces key. In `2026-10-21`, read `scope`: a workspace-bound key (Default Workspace included) has the real ID in `scope.workspace_id`; an all-workspaces key has `scope` `{"type": "organization"}`. If you treated `null` as the Default Workspace, compare `scope.workspace_id` with the Default Workspace's ID (List Workspaces with `include_default=true`, take the one named `"Default"`).

## SDK and CLI changes (Python and TypeScript)

The removed parameters and old field names are also gone from SDK types and CLI flags.

### `MessageParam` is a union, one type per role

`MessageParam` is now `UserMessageParam | AssistantMessageParam | SystemMessageParam` (and `BetaMessageParam` likewise). The JSON sent is unchanged.

```python
# Python: calling MessageParam(...) now raises TypeError
# Before
message = MessageParam(role="user", content="Hello")

# After
message: MessageParam = {"role": "user", "content": "Hello"}
```

In Python, a dict whose `role` is a variable that can hold more than one role no longer type-checks; build one dict per role. Subclass the per-role type (for example `UserMessageParam`), not the union.

```typescript
// TypeScript: an interface can't extend a union
// Before
interface StoredMessage extends Anthropic.MessageParam {
  id: string;
}

// After
type StoredMessage = Anthropic.MessageParam & { id: string };
```

In TypeScript, the system type's `content` is a string or text blocks (plus `tool_addition` / `tool_removal` in beta), and only it has `clear_at` and `output_config` in beta; check `role` before replacing content. `BetaMessageParam["content"]` can be `undefined`; use `BetaMessageContentParam` for user or assistant content.

### `betas` removed from the non-beta Models methods

```python
# Before
page = client.models.list(betas=[beta])

# After
page = client.beta.models.list(betas=[beta])
```

```typescript
// Before
const page = await client.models.list({ betas: [beta] });

// After
const page = await client.beta.models.list({ betas: [beta] });
```

The `ant models` commands no longer take `--beta`; use `ant beta:models`.

### API keys are sent as `Authorization: Bearer`

The SDKs and CLI send the API key as `Authorization: Bearer <key>` instead of `X-Api-Key`. The Claude API accepts either. Update proxies, gateways, middleware, log filters and test stubs that read `X-Api-Key`.

- Python: applies to `api_key` / `ANTHROPIC_API_KEY` on `Anthropic` and `AsyncAnthropic`, and to `AnthropicAWS`. `AnthropicFoundry` still sends `x-api-key`.
- TypeScript: applies to `apiKey` / `ANTHROPIC_API_KEY` and to `@anthropic-ai/aws-sdk`. `@anthropic-ai/foundry-sdk` still sends `X-Api-Key`.

If a proxy needs `X-Api-Key`:

```python
import httpx2
from anthropic import Anthropic, DefaultHttpxClient


def send_x_api_key(request: httpx2.Request) -> httpx2.Request:
    if request.headers.get("Authorization") == f"Bearer {api_key}":
        del request.headers["Authorization"]
        request.headers["X-Api-Key"] = api_key
    return request


client = Anthropic(api_key=api_key, http_client=DefaultHttpxClient(auth=send_x_api_key))
```

```typescript
const client = new Anthropic({
  apiKey,
  defaultHeaders: { "X-Api-Key": apiKey, Authorization: null }
});
```

Per-request overrides: `extra_headers={"X-Api-Key": other_key}` (Python) or `headers: { "X-Api-Key": otherKey }` (TypeScript) no longer replace the client's key; both keys go out. Send `Authorization: Bearer <other_key>` instead. In Python, `extra_headers={"X-Api-Key": omit}` no longer keeps the key off a request; use `{"Authorization": omit}`.

### A client holds one credential

The credential is one of `api_key` / `apiKey`, `auth_token` / `authToken`, `credentials`, `config` or `profile`. Passing two or more raises `ValueError` (Python) or throws `TypeError` (TypeScript). With none passed, the first set of these is used: `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, the profile `ANTHROPIC_PROFILE` names, the workload identity federation variables, the active profile on disk. A warning is logged once when both of the first two are set. `copy()` / `with_options()` / `withOptions()` replace the credential.

```python
client = Anthropic(api_key=key, auth_token=token)
# Before: sent `X-Api-Key: <key>` and `Authorization: Bearer <token>`
# After:  raises `ValueError`
```

TypeScript platform clients (AWS, Bedrock, Foundry, `AnthropicVertex`) now throw an `AnthropicError` for credential options they don't authenticate with.

### Other changes

- **TypeScript tool runner:** client-side `compactionControl` and `@anthropic-ai/sdk/lib/tools/CompactionControl` are removed. Use server-side compaction (`betas: ["compact-2026-01-12"]` plus a `compact_20260112` edit in `context_management`, threshold at least 50,000 tokens), or `runner.compactBeforeNextTurn()`. See [compaction.md](./compaction.md).
- **Text Completions** is removed from the TypeScript, Go, Java and Ruby SDKs and the CLI.
- **License:** the new SDK and CLI versions are Apache 2.0; earlier versions stay MIT. No code changes needed.

## Related

- [SDKs](./sdks.md)
- [Authentication](./authentication.md)
- [Messages API](./messages-api.md)
- [Citations](./citations.md)
- [Migrating to API version 2026-10-21 (official)](https://platform.claude.com/docs/en/api/migration-2026-10-21)
