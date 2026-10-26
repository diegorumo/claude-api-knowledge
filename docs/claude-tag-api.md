# Claude Tag API

> **Last updated:** 2026-10-26  
> **Sources:** platform.claude.com/docs/en/manage-claude/claude-tag-api, platform.claude.com/docs/en/release-notes/overview (Oct 20, 2026)

**Summary:** Beta Admin API (Oct 20, 2026) for reading and changing [Claude Tag](https://claude.com/docs/claude-tag/overview) (Claude in Slack) settings on Slack channels and workspaces, seeing which settings apply in a channel and where each comes from, and attaching Access bundles. Claude Enterprise organizations only; not available to Claude Platform (Claude Console) organizations.

## Requirements

- Claude Enterprise plan with Claude Tag set up and at least one Slack workspace or Enterprise Grid connected.
- An Admin API key that the primary owner creates in claude.ai (**Organization settings > API**) with `read:claude_tag` (all `GET` endpoints), `write:claude_tag` (all `POST` / `DELETE` endpoints), or both. `write` does not grant reads. A key created in the Claude Console does not work.
- Headers: `x-api-key`, `anthropic-version: 2023-06-01`, and `anthropic-beta: experimental-claude-tag-2026-10-21`. Without the beta header, requests return `404` as if the endpoint didn't exist. `anthropic-workspace-id` is ignored.
- SDKs: `client.beta.organization.claude_tag` (Python), `client.beta.organization.claudeTag` (TypeScript); CLI: `ant beta:organization:claude-tag`. They send the version and beta headers for you.

## Endpoints (14)

| Resource         | Endpoints                                                                                                                                                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Slack channels   | `POST /v1/organizations/claude_tag/slack_channels` (set up), `GET .../slack_channels` (list / find by Slack IDs), `GET .../slack_channels/{id}`, `POST .../slack_channels/{id}` (update), `DELETE .../slack_channels/{id}` (remove)          |
| Slack workspaces | `GET /v1/organizations/claude_tag/slack_workspaces`, `GET .../slack_workspaces/{id}`, `POST .../slack_workspaces/{id}` (update)                                                                                                                |
| Bundles          | `GET /v1/organizations/claude_tag/bundles`, `GET .../bundles/{bundle_id}` (read-only)                                                                                                                                                         |
| Bundle targets   | `GET /v1/organizations/claude_tag/bundles/{bundle_id}/targets`, `POST .../targets` (attach), `GET .../targets/{target_id}`, `DELETE .../targets/{target_id}` (detach)                                                                       |

Connecting Slack workspaces or Grids, the organization's Slack defaults, creating or editing Access bundles, spend limits, and any other setting stay in claude.ai.

## Quick start

```python
import anthropic

client = anthropic.Anthropic()  # Admin API key in ANTHROPIC_API_KEY

workspaces = client.beta.organization.claude_tag.slack_workspaces.list()
for workspace in workspaces:
    print(f"{workspace.id}: {workspace.slack.workspace_id}")
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const workspaces = await client.beta.organization.claudeTag.slackWorkspaces.list();
for await (const workspace of workspaces) {
  console.log(`${workspace.id}: ${workspace.slack.workspace_id}`);
}
```

## Settings

Each level (channel, workspace, organization Slack defaults) stores the same five optional settings. A `null` value is not set at that level and is inherited; `sources` shows where an applied value comes from.

| Setting          | Type    | Meaning                                                                                                     |
| ---------------- | ------- | ----------------------------------------------------------------------------------------------------------- |
| `enabled`        | boolean | Whether Claude is on                                                                                        |
| `instructions`   | string  | Standing guidance, at most 8,192 characters after trimming; blank text clears it                           |
| `model`          | string  | Model new sessions start on (at most 128 characters)                                                        |
| `environment_id` | string  | Cloud environment (`env_` prefix); a self-hosted runner pool set in claude.ai shows as `ccpool_` and can't be changed here |
| `guest_policy`   | object  | `{"type": "allow"}`, `{"type": "restrict"}` or `{"type": "channel_only"}`                                    |

## IDs

| ID                       | Prefix                      |
| ------------------------ | --------------------------- |
| Slack channel resource   | `slch_`                     |
| Slack workspace resource | `slws_`                     |
| Bundle                   | `capp_`                     |
| Bundle target            | `btgt_`                     |
| Slack's workspace ID     | `T` (or `E` for a Grid)     |
| Slack's channel ID       | `C` or `G`                  |

In **query parameters**, `slack_channel_id` / `slack_workspace_id` are Slack's own IDs; in **request bodies and responses** they are this API's `slch_` / `slws_` IDs (Slack's own IDs sit under `slack`). Only channels and workspaces that Claude Tag holds a settings record for ("set up") have an `id` and are listed.

## Related

- [Compliance API](./compliance-api.md)
- [Claude Tag API (official)](https://platform.claude.com/docs/en/manage-claude/claude-tag-api)
