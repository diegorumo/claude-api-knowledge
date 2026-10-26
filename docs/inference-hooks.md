# Inference Hooks

> **Last updated:** 2026-10-26  
> **Sources:** platform.claude.com/docs/en/manage-claude/inference-hooks, platform.claude.com/docs/en/release-notes/overview (Oct 20, 2026)

**Summary:** Inference hooks send each governed prompt in a Claude Enterprise organization to an AI security server that the organization (or its security vendor) runs, and wait for an allow or deny verdict before inference runs. Out of beta for Claude Enterprise organizations since Oct 20, 2026. Claude Platform (API) organizations are out of scope.

## How it works

1. A user submits a prompt on a governed surface (claude.ai, Cowork, Claude Code, Claude for Microsoft 365, Claude Tag, Claude in Chrome, and others, on web, desktop, mobile, CLI or Slack).
2. Anthropic sends an HTTPS `POST` with the conversation transcript to your endpoint, signed per [Standard Webhooks](https://www.standardwebhooks.com/) once your organization generates a signing secret.
3. Your server returns a verdict within the configured timeout (5 seconds by default): `{"action": "allow"}`, or a deny with a user-facing `deny_reason`.
4. On deny, the request never reaches the model; the user sees your reason plus a standing admin message, and the denial is logged in the Activity Feed.

Two hook events: `prompt` (once per governed inference request) and `tool_call` (when Claude's response contains tool calls, before any runs; only with **Validate tool calls** on, one verdict per response).

- Your server sees transcript text, tool calls and results, and text extracted from attachments. It never gets raw file or image bytes, system prompts, tool definitions or Anthropic-internal context.
- If your server is unreachable, errors or times out, the failure handling setting decides: block, or allow without inspection. Sustained failures trip a circuit breaker that resets automatically.
- Rollout controls: shadow mode, a rollout percentage, and role exclusions.
- Verdicts are allow or deny only; rewriting or redacting is not supported. Image-only content isn't inspected.
- Not available on Amazon Bedrock or Google Cloud. Configuring requires the `organization:manage` permission (Owner and Primary owner).

## Inference hooks vs. the Compliance API

|              | Inference hooks                         | Compliance API                          |
| ------------ | --------------------------------------- | --------------------------------------- |
| When it acts | Inline, before inference                | After the fact                          |
| What it does | Allows or denies each governed request  | Retrieves activity and content for audit |
| Direction    | Anthropic calls your server             | You call Anthropic's API                |

## Related

- [Compliance API](./compliance-api.md)
- [Inference hooks (official)](https://platform.claude.com/docs/en/manage-claude/inference-hooks)
- [Develop an Inference hooks integration (official)](https://platform.claude.com/docs/en/manage-claude/inference-hooks-endpoint)
