# Message Threads (beta)

> **Last updated:** 2026-11-09  
> **Sources:** platform.claude.com/docs/en/build-with-claude/message-threads, platform.claude.com/docs/en/release-notes/overview (Nov 2, 2026), platform.claude.com/docs/en/build-with-claude/refusals-and-fallback  
> **Status:** Beta since Nov 2, 2026. Beta header: `message-threads-2026-08-12`

**Summary:** A message thread keeps a Messages API conversation on the server for a short time. You send the full conversation (with `system` and `tools`) once, then on each later turn send only the new messages plus the `id` of the previous response. Requests get smaller and time to first token stays lower and steadier as the conversation grows. Most useful for long agent loops.

Message threads are not the same as [session threads](./managed-agents.md) in Claude Managed Agents.

## Availability

| Item           | Value                                                                                                                  |
| -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Beta header    | `message-threads-2026-08-12` (send it on every request of a thread)                                                    |
| Endpoint       | `POST /v1/messages` with the `thread` field                                                                            |
| Platforms      | Claude API, Claude Platform on AWS, Claude Models on Google Cloud (Anthropic managed), Microsoft Foundry (all beta)    |
| Models         | All current and legacy models listed on the page, from `claude-fable-5-1` to `claude-sonnet-4-5-20250929` ("beta on every model") |
| Pricing        | No change to pricing or rate limits                                                                                    |
| ZDR / HIPAA    | Not eligible. Held in memory, encrypted under a server-managed key, never written to durable storage                   |

## How it works

- **Create.** Send the first request as usual, with the full conversation, `system` and `tools`, and add `"thread": {"type": "create"}`. The API stores the conversation (including Claude's reply) under the response `id`.
- **Continue.** Send only the messages since the previous response and `"thread": {"type": "continue", "previous_message_id": "<previous id>"}`. Leave `system` and `tools` out; the thread keeps them. Use each new response `id` for the next turn.
- **Replay.** Threads are best-effort. A thread expires **5 minutes** after its last request and is deleted **no more than 1 hour** after its last use. Each stored response restarts the 5 minutes. If the thread can't be continued, a continue request returns **404 `thread_not_found`**: send the full conversation again in a new create request.

Your application stays the source of truth. Keep your own copy of every message and build the replay path from the start: no response says whether the API stored that turn; you find out on the next continue request.

A thread is append-only, and each response `id` can be continued **once**. To edit or branch an earlier message, or to compact, start a new thread with a create request. There is no API to list, read or delete a thread.

## SDK thread helper

The helper builds the thread requests for you, adds the beta header, keeps the full conversation in memory, and replays on `thread_not_found`, `thread_already_continued` or `thread_continue_in_flight`. (Python and TypeScript examples copied from the official page.)

```python
import anthropic

client = anthropic.Anthropic()

# Create the helper. This call sends no request.
thread = client.beta.messages.thread(
    model="claude-opus-5-5",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    system="You are a support agent for Acme. Keep replies short.",
)

# Turn 1: the helper creates a thread and sends the full conversation
thread.send({"role": "user", "content": "Hi! I have a question about an order."})

# Turn 2: the helper continues the thread and sends only the new message
second = thread.send(
    {"role": "user", "content": "How long does standard shipping usually take?"}
)

for block in second.content:
    if block.type == "text":
        print(block.text)
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

// Create the helper. This call sends no request.
const thread = client.beta.messages.thread({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  system: "You are a support agent for Acme. Keep replies short.",
});

// Turn 1: the helper creates a thread and sends the full conversation
await thread.send({ role: "user", content: "Hi! I have a question about an order." });

// Turn 2: the helper continues the thread and sends only the new message
const second = await thread.send({
  role: "user",
  content: "How long does standard shipping usually take?",
});

for (const block of second.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

Other helper methods (Python / TypeScript names): `thread.stream(...)`, `thread.messages`, `thread.previous_message_id` / `previousMessageId`, `thread.append(...)`, `thread.update(...)`, `thread.fork()`, `thread.params`, `thread.parse(...)` (structured output), `thread.tool_runner()` / `toolRunner()`. The SDK's beta tool runner uses a thread **by default**; turn it off with `tool_runner(..., thread=False)` / `toolRunner({ ..., thread: false })`.

Create the helper with `unsupported_request_behavior="continue_unthreaded"` (TypeScript `unsupportedRequestBehavior`) to fall back to sending without `thread` after a `thread_unsupported_request` error, or when a continue right after a replay returns `thread_not_found` again. Without it, the helper replays on every turn. A tool runner that creates its own thread uses that setting by default.

What the helper does **not** do: retry a continue request after a dropped stream (it keeps finished blocks and replays on the next turn; call `send()` with no messages to re-send the interrupted turn), and it surfaces `thread_fingerprint_mismatch` / `thread_unsupported_request` errors to your code.

## Sending the `thread` field yourself

```python
first = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    system="You are a support agent for Acme. Keep replies short.",
    messages=[{"role": "user", "content": "Hi! I have a question about an order."}],
    thread={"type": "create"},
    betas=["message-threads-2026-08-12"],
)

# Leave out system and tools: the thread keeps them.
second = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},
    messages=[
        {"role": "user", "content": "How long does standard shipping usually take?"}
    ],
    thread={"type": "continue", "previous_message_id": first.id},
    betas=["message-threads-2026-08-12"],
)
```

```typescript
const first = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  system: "You are a support agent for Acme. Keep replies short.",
  messages: [{ role: "user", content: "Hi! I have a question about an order." }],
  thread: { type: "create" },
  betas: ["message-threads-2026-08-12"],
});

const second = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  cache_control: { type: "ephemeral" },
  messages: [{ role: "user", content: "How long does standard shipping usually take?" }],
  thread: { type: "continue", previous_message_id: first.id },
  betas: ["message-threads-2026-08-12"],
});
```

### What to send on each request

| Group       | Fields                                                                                                                                                                                           | Rule                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| Create-only | `system`, `tools`                                                                                                                                                                                | Create request only. Sending either on a continue request, even as `null`, returns 400.                               |
| Fixed       | `model`, `tool_choice`, `thinking`, `mcp_servers` (except each server's auth token), `context_management`, `output_config`, the `skills` in `container`, top-level `cache_control`, the set of `anthropic-beta` values | Same values on every request. A difference returns 400 `thread_fingerprint_mismatch`, naming the fields.              |
| Free        | `max_tokens`, `stop_sequences`, `stream`, `metadata`, `service_tier`, and other fields not named                                                                                                 | Can change turn to turn.                                                                                              |

- The API does not store MCP server URLs, MCP auth tokens or the container `id`: send `mcp_servers` and `container` on every request that uses them.
- To change a create-only or fixed field, start a new thread.
- On Fable 5.1, Mythos 5.1, Opus 5.5, Sonnet 5.5 and Haiku 5.5, changing `system` or `tools` invalidates earlier `thinking` / `redacted_thinking` blocks and the create request can return 400. Leave those blocks out of the replayed conversation to avoid it.
- Send **only** new messages on a continue request; the API doesn't de-duplicate. A continue that starts with the previous response returns an error.
- After `stop_reason: "pause_turn"`, resume with a continue request and `"messages": []`. Don't send a new `user` message while paused.

## Prompt caching on a thread

A thread does not cache for you. Send a top-level `cache_control` on every request to cache each turn (it's a fixed field). A request with `thread` can carry **at most 4** `cache_control` breakpoints; the top-level one counts, and breakpoints on the create request's `system` / `tools` count again on every continue. One too many returns 400 `thread_unsupported_request`. Breakpoints can't be moved or removed once sent. See [Prompt Caching](./prompt-caching.md).

## Streaming

The response `id` arrives in `message_start`, but continue from it only after `message_stop` (earlier returns 404). If a continue stream drops before `message_stop`, send the same continue again; if that returns `thread_already_continued` or `thread_continue_in_flight`, replay in a new create request.

## Refusals and fallbacks

- After `stop_reason: "refusal"`, do not continue the thread. Start a new thread that leaves out or rephrases the turn that led to the refusal. The helper does this.
- [Server-side fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#server-side-fallback) works with threads on the Claude API. Send `fallbacks` on every request of a thread or on none (adding/dropping it returns `thread_fingerprint_mismatch`). The API doesn't store a turn written by a fallback model: if `usage.iterations` has a `fallback_message` entry, send the full conversation in a new create request (keep the `server-side-fallback-2026-07-01` beta header), either repeating the same request or starting a new thread on the fallback model without `fallbacks`.

## Errors

These are returned before the model runs; the request uses no tokens and the thread's expiry doesn't change. The code is in `error.details.error_code`.

| Status | `error_code`                  | Meaning                                                                           | What to do                                                             |
| ------ | ----------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 404    | `thread_not_found`            | The API can't continue the thread                                                 | Send the full conversation in a new create request                     |
| 400    | `thread_fingerprint_mismatch` | A fixed field differs from the stored value (message names the fields)            | Keep fixed fields the same, or start a new thread                      |
| 400    | `thread_already_continued`    | Each response `id` can be continued once                                          | Continue from the latest `id`; replay if you never received it         |
| 400    | `thread_continue_in_flight`   | Another continue from this `id` is still running                                  | Let it finish and continue from its `id`, or replay                    |
| 400    | `thread_unsupported_request`  | A feature or org setting threads don't support (`error.details.unsupported` names it) | Remove the feature, or send without `thread`                       |

`thread_not_found` causes include: expiry; the previous turn was served but not stored (for example a response ending in a tool call cut off by `max_tokens`); a fallback model wrote it; it's still generating; a different organization, workspace, user or service account (a thread created with a legacy workspace key stays with that key); or new content the stored conversation can't take (for example the first image in a conversation that had none). A request for a non-existent model also returns 404, so check `error_code` before replaying.

`error.details.unsupported` values: `context_management`, `cache_control`, `fallbacks`, `hipaa`, `inference_hooks` (more may be added).

## Not supported

- Message Batches (a batch item with `thread` ends with `invalid_request_error`)
- Token counting (400; count the full conversation instead)
- Compaction on demand (`compaction` parameter, or a `compaction` block with `compact-2026-09-04`, returns 400)
- Context editing, except `clear_thinking_20251015` with `keep: "all"`
- Organizations with HIPAA readiness enabled, or with [Inference hooks](./inference-hooks.md) turned on

## Data retention

A thread holds the conversation sent to the model, the create request's `system` and `tools`, and a fingerprint of the fixed fields. Files are held by reference, so keep every file the conversation uses available while you use the thread.

## Related

- [Messages API](./messages-api.md)
- [Prompt Caching](./prompt-caching.md)
- [Streaming](./streaming.md)
- [Compaction](./compaction.md)
