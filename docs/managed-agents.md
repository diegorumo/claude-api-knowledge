# Managed Agents (Beta)

> **Last updated:** 2026-08-17  
> **Status:** Beta — active development  
> **SDK changelog:** v0.100.0+ (May 2026), v0.115.0 (June 2026), v0.116.0–v0.122.0 Python / v0.110.0–v0.117.1 TypeScript (July–Aug 2026)

## Overview

The Managed Agents API provides persistent, configurable AI agents with sessions, memory, environments, and tool integrations.

**Key components:**
- **Agents** — Persistent agent definitions with tools, skills, and model config
- **Sessions** — Conversation contexts pinned to an agent
- **Environments** — Execution sandboxes (cloud or self-hosted)
- **Vaults** — Secure credential storage for MCP servers
- **Memory Stores** — Persistent memory across sessions

## Basic Agent Lifecycle

```python
import anthropic

client = anthropic.Anthropic()

# 1. Create an environment
environment = client.beta.environments.create()

# 2. Create an agent
agent = client.beta.agents.create(
    model="claude-sonnet-4-6",
    name="my-assistant",
    system_prompt="You are a helpful assistant.",
    tools=[{"type": "agent_toolset_20260401"}],
)

# 3. Create a session
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
)

# 4. Send a message and stream events
client.beta.sessions.events.send(
    session_id=session.id,
    event={"type": "user.message", "content": "Hello!"},
)

for event in client.beta.sessions.events.stream(session_id=session.id):
    print(event.model_dump_json())
    if event.type == "session.status_idle":
        break
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const environment = await client.beta.environments.create({});
const agent = await client.beta.agents.create({
  model: 'claude-sonnet-4-6',
  name: 'my-agent',
});

const session = await client.beta.sessions.create({
  agent_id: agent.id,
  environment_id: environment.id,
});

await client.beta.sessions.events.send(session.id, {
  event: { type: 'user.message', content: 'Hello!' },
});

const stream = client.beta.sessions.events.stream(session.id);
for await (const event of stream) {
  console.log(JSON.stringify(event));
  if (event.type === 'session.status_idle') break;
}
```

## Handling Custom Tool Calls in Sessions

```python
def handle_session(client, session_id, custom_tools):
    while True:
        for event in client.beta.sessions.events.stream(session_id=session_id):
            if event.type == "agent.custom_tool_use":
                result = custom_tools[event.tool_name](**event.tool_input)
                client.beta.sessions.events.send(
                    session_id=session_id,
                    event={
                        "type": "user.custom_tool_result",
                        "tool_use_id": event.tool_use_id,
                        "content": str(result),
                    },
                )
            elif event.type == "session.status_idle":
                if event.stop_reason.type == "end_turn":
                    return
                break
            elif event.type == "agent.message":
                for block in event.content:
                    if block.type == "text":
                        print(f"Agent: {block.text}")
```

## Session Event Types

| Event Type | Direction | Description |
|-----------|-----------|-------------|
| `user.message` | → API | Send a message to the agent |
| `user.custom_tool_result` | → API | Return custom tool result |
| `agent.message` | ← API | Agent's text response |
| `agent.custom_tool_use` | ← API | Agent requesting your custom tool |
| `agent.thinking` | ← API | Extended thinking block |
| `session.status_idle` | ← API | Session is idle (turn complete) |
| `session.status_running` | ← API | Session is processing |
| `error` | ← API | Error occurred |

## Agent with MCP and Custom Tools

```python
vault = client.beta.vaults.create(name="credentials")
credential = client.beta.vaults.credentials.create(
    vault_id=vault.id,
    type="bearer",
    token="github-token-here",
)

agent = client.beta.agents.create(
    model="claude-sonnet-4-6",
    name="github-agent",
    tools=[
        {"type": "agent_toolset_20260401"},
        {
            "type": "mcp",
            "server": {"type": "url", "url": "https://github-mcp.example.com/sse", "name": "github"},
            "vault_id": vault.id,
            "credential_id": credential.id,
        },
        {
            "name": "get_weather",
            "description": "Get current weather",
            "input_schema": {"type": "object", "properties": {"location": {"type": "string"}}, "required": ["location"]},
        },
    ],
)
```

## Agent Versioning

```python
agent = client.beta.agents.create(model="claude-sonnet-4-6", name="my-agent", tools=[...])

# Update bumps version
agent_v2 = client.beta.agents.update(agent_id=agent.id, skills=["my-skill-id"])
print(f"Version: {agent_v2.version}")  # 2

# Pin session to specific version
session = client.beta.sessions.create(
    agent_id=agent.id,
    agent_version=1,
    environment_id=environment.id,
)
```

## Memory Stores

```python
memory = client.beta.memory_stores.create(name="user-preferences")
client.beta.memory_stores.memories.create(
    memory_store_id=memory.id,
    content="User prefers concise responses.",
)

session = client.beta.sessions.create(
    agent_id=agent.id,
    memory_store_ids=[memory.id],
)
```

## Self-Hosted Sandbox Worker

```typescript
import { betaAgentToolset20260401 } from '@anthropic-ai/sdk/tools/agent-toolset/node';

await client.beta.environments.work.worker({
  environmentId: process.env.ANTHROPIC_ENVIRONMENT_ID!,
  environmentKey: process.env.ANTHROPIC_ENVIRONMENT_KEY!,
  workdir: '/workspace',
  tools: (ctx) => betaAgentToolset20260401(ctx),
}).run(AbortSignal.timeout(3600000));
```

**Standard toolset includes:** `bash`, `read`, `write`, `edit`, `glob`, `grep` (requires Node 22+)

## Event Delta Streaming (v0.115.0+)

Python SDK v0.115.0 / TypeScript SDK v0.109.0 added **event delta streaming** — partial updates streamed as session events progress, so you get incremental content before a full block is complete.

```python
# Delta events appear as streaming counterparts to full events
for event in client.beta.sessions.events.stream(session_id=session.id):
    if event.type == "agent.message.delta":
        # Partial text as it streams
        for delta in event.content_deltas:
            if delta.type == "text_delta":
                print(delta.text, end="", flush=True)
    elif event.type == "agent.message":
        # Full message on completion
        pass
    elif event.type == "session.status_idle":
        break
```

Delta event types mirror the full event types with a `.delta` suffix. Use delta events for real-time UI updates; use full events for reliable state transitions.

## Agent Overrides (v0.115.0+)

Per-session **agent overrides** let you temporarily modify agent behavior without updating the agent definition. Overrides apply only to a single session.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    agent_overrides={
        "model": "claude-sonnet-4-6",      # Override the model
        "system_prompt": "Focus on brevity.",  # Override the system prompt
        "max_tokens": 512,                 # Override max output tokens
    },
)
```

Overrides are useful for A/B testing, user-specific customizations, or temporarily adjusting behavior without modifying the canonical agent definition.

## Reverse Pagination (v0.115.0+)

List endpoints now support reverse pagination to retrieve most-recent items first:

```python
# List sessions newest-first
sessions = client.beta.sessions.list(
    agent_id=agent.id,
    order="desc",  # "asc" (default) or "desc"
    limit=20,
)
for session in sessions.data:
    print(session.id, session.created_at)
```

## Vault Credential Injection Scoping (v0.115.0+)

Vault credentials can now be scoped to specific MCP servers, preventing credentials from being injected into unintended tool calls:

```python
agent = client.beta.agents.create(
    model="claude-sonnet-4-6",
    name="scoped-agent",
    tools=[
        {
            "type": "mcp",
            "server": {"type": "url", "url": "https://github-mcp.example.com/sse", "name": "github"},
            "vault_id": vault.id,
            "credential_id": credential.id,
            "credential_scope": ["github"],  # Only inject for this MCP server
        },
    ],
)
```

## Agent and Deployment Webhook Events (v0.115.0+)

Webhook callbacks can now be configured for both agent events and deployment events:

```python
# Configure webhooks when creating an agent
agent = client.beta.agents.create(
    model="claude-sonnet-4-6",
    name="webhook-agent",
    webhooks=[
        {
            "url": "https://your-app.example.com/webhooks/agent",
            "events": ["session.completed", "session.error", "agent.message"],
        }
    ],
)

# Configure webhooks for a deployment
deployment = client.beta.deployments.create(
    agent_id=agent.id,
    webhooks=[
        {
            "url": "https://your-app.example.com/webhooks/deployment",
            "events": ["deployment.started", "deployment.stopped"],
        }
    ],
)
```

Webhook payloads use the same event format as the streaming API. Implement idempotent handlers — webhooks may be delivered more than once.

## Memory Stores (v0.116.0+ — requires `agent-memory-2026-07-22` beta header)

As of Python SDK v0.116.0 / TypeScript SDK v0.110.0, Memory Stores API operations require the **`agent-memory-2026-07-22`** beta header. The SDK automatically adds this header for all `memory_stores` API calls.

```python
# Memory stores now require the agent-memory beta — SDK handles this automatically
memory = client.beta.memory_stores.create(name="user-preferences")

# Explicitly pass beta if using raw HTTP:
# anthropic-beta: agent-memory-2026-07-22

client.beta.memory_stores.memories.create(
    memory_store_id=memory.id,
    content="User prefers concise responses.",
)

# Attach to a session
session = client.beta.sessions.create(
    agent_id=agent.id,
    memory_store_ids=[memory.id],
)
```

Memory versions are also supported — each update to a memory creates a new version, accessible via `client.beta.memory_stores.memory_versions`.

## Deployments (v0.109.0+)

Python SDK v0.109.0 added support for **Managed Agents deployments** — a way to run agent workloads using environment variable credentials without passing API keys explicitly in code.

```python
import anthropic

# Credentials resolved from env: ANTHROPIC_API_KEY + deployment-specific vars
client = anthropic.Anthropic()

# Create a deployment-backed session
session = client.beta.sessions.create(
    agent_id="agent_...",
    environment_id=os.environ["ANTHROPIC_ENVIRONMENT_ID"],
)

for event in client.beta.sessions.events.stream(session_id=session.id):
    if event.type == "agent.message":
        for block in event.content:
            if block.type == "text":
                print(block.text)
    elif event.type == "session.status_idle":
        break
```

Deployment configurations are defined server-side; the client discovers them via the API rather than specifying them inline. This allows ops teams to manage model, tools, and environment separately from application code.

## Dreams API (Python v0.117.0+ / TypeScript v0.111.0+ — Beta)

Dreams are **asynchronous memory-consolidation jobs** that read a memory store plus a set of session transcripts and write consolidated memories into a new output memory store. Use them to periodically distill long conversation histories into durable memory.

**Endpoints** (both Python and TypeScript as of 2026-07-16):

| Method | Endpoint | Description |
|--------|----------|-------------|
| `create` | `POST /v1/dreams?beta=true` | Start a new dream job |
| `retrieve` | `GET /v1/dreams/{dream_id}?beta=true` | Poll a dream's status |
| `list` | `GET /v1/dreams?beta=true` | List dreams (paginated) |
| `archive` | `POST /v1/dreams/{dream_id}/archive?beta=true` | Archive a completed dream |
| `cancel` | `POST /v1/dreams/{dream_id}/cancel?beta=true` | Cancel a running dream |

**Status lifecycle:** `pending → running → completed | failed | canceled`

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

// Create a dream job: consolidate sessions into a memory store
const dream = await client.beta.dreams.create({
  inputs: [
    { type: 'memory_store', memory_store_id: 'ms_...' },
    { type: 'sessions', session_ids: ['sess_...', 'sess_...'] },
  ],
  model: 'claude-sonnet-4-6',
  instructions: 'Consolidate facts about the user's preferences.',
  betas: ['agent-memory-2026-07-22'],
});

console.log(dream.id, dream.status); // 'dream_...' 'pending'

// Poll until complete
let result = await client.beta.dreams.retrieve(dream.id, { betas: ['agent-memory-2026-07-22'] });
while (result.status === 'pending' || result.status === 'running') {
  await new Promise(r => setTimeout(r, 2000));
  result = await client.beta.dreams.retrieve(dream.id, { betas: ['agent-memory-2026-07-22'] });
}

console.log(result.status, result.outputs, result.usage);
```

**BetaDream object:**
- `id` — dream ID (`dream_...`)
- `status` — `'pending' | 'running' | 'completed' | 'failed' | 'canceled'`
- `inputs` — array of `BetaDreamMemoryStoreInput | BetaDreamSessionsInput`
- `outputs` — array of `BetaDreamOutput` (memory store references)
- `model` — `BetaDreamModelConfig`
- `error` — `BetaDreamError | null`
- `usage` — `{ input_tokens, output_tokens, cache_creation_input_tokens, cache_read_input_tokens }`
- `created_at`, `ended_at`, `archived_at`

**List with filters (TypeScript):**
```typescript
const dreams = await client.beta.dreams.list({
  statuses: ['completed'],
  include_archived: false,
  betas: ['agent-memory-2026-07-22'],
});
```

**List with filters (Python):**
```python
dreams = client.beta.dreams.list(
    statuses=["completed"],
    include_archived=False,
    created_at_gt="2026-07-01T00:00:00Z",  # filter by date (Python-only parameter)
    betas=["agent-memory-2026-07-22"],
)
```

> **Note:** Dreams require the `agent-memory-2026-07-22` beta header (same as Memory Stores). Both Python (v0.117.0+) and TypeScript (v0.111.0+) SDKs support the Dreams API. Python's `list()` also accepts `created_at_gt` and `created_at_lt` datetime filters. Claude Opus 5 is supported as of Aug 1, 2026; Claude Fable 5 and Sonnet 5 as of Jul 10, 2026.

### Dreams `output_behavior` (Python v0.122.0 / TypeScript v0.117.0)

The `create` call now accepts an `output_behavior` parameter that controls where consolidated memories are written:

```python
# Create a new memory store for the output (default behavior)
dream = client.beta.dreams.create(
    inputs=[...],
    model="claude-opus-5",
    output_behavior="create_new",   # creates a fresh memory store
    betas=["agent-memory-2026-07-22"],
)

# OR: update the input memory store in place
dream = client.beta.dreams.create(
    inputs=[...],
    model="claude-opus-5",
    output_behavior="update_input", # overwrites the input memory store
    betas=["agent-memory-2026-07-22"],
)
```

`output_behavior` values: `"create_new"` (default) | `"update_input"`

## Model Effort (Python v0.118.0+ / TypeScript v0.113.0+)

The `BetaManagedAgentsModelConfig` object (used when creating/updating an agent) now accepts an **`effort`** field that controls how hard Claude works on each turn. It sets `output_config.effort` on every Messages API call the session makes.

**Effort levels** (five tiers):
- `BetaManagedAgentsEffortLow`
- `BetaManagedAgentsEffortMedium`
- `BetaManagedAgentsEffortHigh`
- `BetaManagedAgentsEffortXhigh`
- `BetaManagedAgentsEffortMax`

The model config also accepts a **`speed`** field (`"standard"` or `"fast"`). `"fast"` provides significantly faster output token generation at premium pricing; not all models support it.

```python
agent = client.beta.agents.create(
    model={
        "id": "claude-sonnet-4-6",
        "effort": {"type": "high"},   # sets output_config.effort on every turn
        "speed": "standard",
    },
    name="my-agent",
)
```

## Pinned Inference Geo (Python v0.122.0+ / TypeScript v0.117.0+)

Control where model inference runs by setting `inference_geo` inside the `model` object when creating an agent. This is useful for data residency and compliance requirements.

```python
# Pin inference to US region
agent = client.beta.agents.create(
    name="US-only agent",
    model={
        "id": "claude-opus-5",
        "inference_geo": "us",   # "us" or "global"
    },
    system="You are a helpful assistant.",
)
print(agent.model.inference_geo)  # "us"
```

```typescript
const agent = await client.beta.agents.create({
  name: "US-only agent",
  model: { id: "claude-opus-5", inference_geo: "us" },
  system: "You are a helpful assistant.",
});
console.log(agent.model.inference_geo); // "us"
```

**`inference_geo` values:** `"us"` | `"global"` | (omit to use workspace default)

**Important constraints:**
- The pin is validated against the workspace's `allowed_inference_geos` at save time, session creation, and on every turn
- If the workspace allowlist narrows after a pin is set, new sessions cannot be created and running sessions refuse further turns
- Setting a pin on a model that doesn't support geographic inference pinning returns 400
- In multiagent configurations, all roster members must use the same `inference_geo` or all must be unset
- To clear a pin, update the agent's `model` object without `inference_geo`
- Can also be set per-session as an override (see agent-setup docs)

See also [Data residency docs](https://platform.claude.com/docs/en/manage-claude/data-residency).

## Initial Session Events (Python v0.118.0+ / TypeScript v0.113.0+)

The `sessions.create()` call now accepts an **`initial_events`** parameter — an ordered list of events to process at session creation. This eliminates a round-trip for sessions that need to start with a user message or an outcome definition.

**Supported event types:**
- `user.message`
- `user.define_outcome`

**Maximum:** 50 initial events.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    initial_events=[
        {
            "type": "user.message",
            "content": "Summarize the attached document.",
        }
    ],
)
# Session starts processing immediately — no need to call events.send() first
```

```typescript
const session = await client.beta.sessions.create({
  agent_id: agent.id,
  environment_id: environment.id,
  initial_events: [
    { type: 'user.message', content: 'Summarize the attached document.' },
  ],
});
```

## Threads Delta Streaming (Python v0.118.0+ / TypeScript v0.113.0+)

Subagent threads now expose their own event stream, letting you follow a specific thread independently of the parent session stream. Use `client.beta.sessions.threads.events.stream()`.

**Endpoint:** `GET /v1/sessions/{session_id}/threads/{thread_id}/stream`

```python
# List threads for a session
threads = client.beta.sessions.threads.list(session_id=session.id)
for thread in threads.data:
    print(thread.id, thread.status)  # BetaManagedAgentsSessionThreadStatus

# Stream events from a specific thread
for event in client.beta.sessions.threads.events.stream(
    thread_id=thread.id,
    session_id=session.id,
    event_deltas=True,   # enable delta streaming for this thread
):
    print(event.type, event.model_dump_json())
```

```typescript
// Stream events from a specific thread
const threadStream = client.beta.sessions.threads.events.stream(
  thread.id,
  { session_id: session.id, event_deltas: true }
);
for await (const event of threadStream) {
  console.log(event.type, JSON.stringify(event));
}
```

**Thread status lifecycle:** `running → idle → rescheduling | terminated`

**Thread endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `threads.retrieve()` | `GET /v1/sessions/{id}/threads/{thread_id}` | Get a thread |
| `threads.list()` | `GET /v1/sessions/{id}/threads` | List threads (paginated) |
| `threads.archive()` | `POST /v1/sessions/{id}/threads/{thread_id}/archive` | Archive a thread |
| `threads.events.list()` | `GET /v1/sessions/{id}/threads/{thread_id}/events` | List thread events |
| `threads.events.stream()` | `GET /v1/sessions/{id}/threads/{thread_id}/stream` | Stream thread events |

**Key types:**
- `BetaManagedAgentsSessionThread` — thread object (id, session_id, status, stats, usage)
- `BetaManagedAgentsSessionThreadStatus` — `"running" | "idle" | "rescheduling" | "terminated"`
- `BetaManagedAgentsStreamSessionThreadEvents` — union of all thread event types
- `BetaManagedAgentsSessionThreadCreatedEvent` — fired when a subagent spawns a new thread
- `BetaManagedAgentsSessionThreadStatusRunningEvent` / `...IdleEvent` / `...TerminatedEvent` / `...RescheduledEvent`

## Session Tool Call Permissions — `evaluated_permission` (TypeScript v0.111.0+)

Session tool-use events (`agent.tool_use`, `agent.custom_tool_use`, `agent.mcp_tool_use`) now carry an **`evaluated_permission`** field that gates execution:

| Value | Behavior |
|-------|----------|
| `"allow"` | Tool runs immediately |
| `"ask"` | Execution held; must send `user.tool_confirmation` to proceed |
| `"deny"` | Tool is not executed; call still visible to consumers |
| *(absent)* | Treated as `"allow"` (default) |

**Tool confirmation flow:**
```typescript
const stream = client.beta.sessions.events.stream(session.id);
for await (const event of stream) {
  if (event.type === 'agent.tool_use' && event.evaluated_permission === 'ask') {
    // Show user a confirmation prompt
    const approved = await promptUser(`Allow tool "${event.name}"?`);
    await client.beta.sessions.events.send(session.id, {
      event: {
        type: 'user.tool_confirmation',
        tool_use_id: event.id,
        result: approved ? 'allow' : 'deny',
        deny_message: approved ? undefined : 'User declined.',
      },
    });
  }
}
```

**`user.tool_confirmation` event fields:**
- `tool_use_id` (required) — matches the `agent.tool_use` event's `id`
- `result` (required) — `'allow'` or `'deny'`
- `deny_message` (optional) — context string, only valid when `result === 'deny'`
- `session_thread_id` (optional) — routes to a subagent thread

**Fail-closed semantics:** An unrecognized `evaluated_permission` value is treated as `"ask"` (held). An unrecognized confirmation verdict is treated as `"deny"`.

**SDK helper — `SessionToolRunner`:** The TypeScript SDK's `SessionToolRunner` helper (used internally by `client.beta.sessions.toolRunner(...)`) handles `evaluated_permission` automatically:
- `"allow"` calls execute immediately
- `"ask"` calls are held in a queue until `user.tool_confirmation` arrives
- `"deny"` calls are yielded with `{ confirmation: 'deny', posted: false }` but not executed
- Idle bounding: the runner stops after `maxIdleMs` (default 60 s) of inactivity once the session reaches `stop_reason: { type: "end_turn" }`. The countdown is paused while any tool confirmation is outstanding.

## Session Budgets (Python v0.121.0+ / TypeScript v0.116.0+)

Set a **hard spend ceiling** on a session so it stops issuing new model requests once the tracked cost reaches the limit.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    budget_limit={
        "type": "limit",
        "max_list_cost": {
            "amount": "10.00",
            "currency": "USD",
        },
    },
)
```

When the session's accumulated list cost reaches `max_list_cost`, no further model requests are issued. The session enters idle state; you can check usage stats and decide whether to extend or end the session.

**`BetaManagedAgentsBudgetLimit` fields:**
- `type` (required): `"limit"`
- `max_list_cost` (`BetaMonetaryAmount`): monetary threshold with `amount` (decimal string) and `currency` (ISO 4217 code)

## Advisor Tool (Python v0.121.0+ / TypeScript v0.116.0+)

The **advisor tool** lets the session's primary thread consult a second Claude model mid-turn — useful for peer review, fact-checking, or specialized reasoning without a full subagent thread.

### Adding to a tool definition

```python
agent = client.beta.agents.create(
    model="claude-sonnet-4-6",
    name="advised-agent",
    tools=[
        {"type": "agent_toolset_20260401"},
        {
            "type": "advisor_20260301",  # advisor tool type string
            "name": "advisor",           # must be "advisor"
            "model": "claude-opus-5",    # model to consult
            "max_tokens": 1024,          # optional: cap advisor output
            "max_uses": 5,               # optional: cap invocations per request
        },
    ],
)
```

### Session roster entry

Alternatively, configure the advisor at the session level via the advisor roster:

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    advisor={
        "type": "advisor",
        "model": "claude-opus-5",    # model to consult
    },
)
```

**`BetaManagedAgentsAdvisor` / `BetaManagedAgentsAdvisorParams` fields:**
- `type` (required): `"advisor"`
- `model` (required): Claude model ID; must be permitted as an advisor for the session's primary model

**Result blocks:**  
The primary thread receives `BetaAdvisorResultBlock` and `BetaAdvisorRedactedResultBlock` content blocks in response to advisor invocations.

**Constraints:**
- Maximum **one** advisor per session roster
- Reserved roster name: `anthropic.advisor`
- Advisor model must be permitted by the primary model's policy

## Skills in Sessions (Python v0.121.0+ / TypeScript v0.116.0+)

Sessions can load skills at creation time. See [Skills API](./skills-api.md) for how to upload custom skills.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    skills=[
        # Anthropic-managed skill
        {"type": "anthropic", "skill_id": "xlsx"},
        # Your custom skill
        {"type": "custom", "skill_id": "skill_01XJ5...", "version": "1.0.0"},
    ],
)
```

**`BetaManagedAgentsAnthropicSkill` fields:** `type: "anthropic"`, `skill_id` (e.g. `"xlsx"`), `version` (optional, default latest)  
**`BetaManagedAgentsCustomSkill` fields:** `type: "custom"`, `skill_id` (e.g. `"skill_01XJ5..."`), `version` (optional, default latest)

## GitHub Repository Resource (Python v0.121.0+ / TypeScript v0.116.0+)

Sessions can auto-load skills from a GitHub repository. Configure the repository as a resource on the environment or pass it at session creation via `BetaManagedAgentsGitHubRepositoryResourceConfig`.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    github_repository={
        # GitHub repository config fields (resolved server-side)
    },
)
```

See [Skills API](./skills-api.md) for full GitHub auto-loading details.

## Gotchas

- Managed Agents is under active development — check SDK changelogs for new features
- Sessions have inactivity timeouts — implement keep-alive or reconnection logic
- Custom tool results must be returned promptly or the session may timeout
- v0.103.0+: self-hosted sandboxes supported
- v0.109.0+: deployments with environment variable credentials supported
- v0.112.0+: memory tool now correctly creates parent directories with proper permissions
- v0.115.0+: agent overrides, reverse pagination, vault scoping, webhooks available
- v0.116.0+ / TypeScript v0.110.0+: Memory Stores require the `agent-memory-2026-07-22` beta header (SDK adds automatically)
- TypeScript v0.111.0+ / Python v0.117.0+: Dreams API and `evaluated_permission` on session tool calls
- Python v0.118.0+ / TypeScript v0.113.0+: `initial_events` on `sessions.create()`, `effort` on model config, threads event streaming
- Python v0.119.0+: binary file handling fixed in agent toolset read/edit operations
- Python v0.121.0+ / TypeScript v0.116.0+: session budgets (`budget_limit`), advisor tool, skills in sessions, GitHub repository resource
- Python v0.122.0+ / TypeScript v0.117.0+: Dreams `output_behavior` param; inference_geo pinning; `beta.messages.parse/stream/tool_runner` now available in Bedrock and Vertex clients; symlink loop rejection in skill archives
- `claude-opus-4-1` / `claude-opus-4-1-20250805` formally removed and retired (Aug 5, 2026) — migrate to claude-opus-5 or newer
- Webhook handlers must be idempotent — events may be delivered more than once
- Agent overrides are session-scoped only — they do not persist to the agent definition
- Python Dreams `list()` accepts `created_at_gt`/`created_at_lt` datetime filters; TypeScript does not yet
- `model_context_window_exceeded` stop reason is now possible — handle it by trimming old messages and retrying
- Budget limits stop new model requests but do not interrupt a turn already in progress
- `inference_geo` pin validated against workspace `allowed_inference_geos` on every turn; if workspace restricts geo after pin set, running sessions refuse further turns
- In multiagent configurations, all roster members must share the same `inference_geo` value or all be unset

## Related

- [Tool Use](./tool-use.md)
- [Skills API](./skills-api.md)
- [MCP Integration](./mcp.md)
- [Files API](./files-api.md)
- [Agent Patterns](./agent-patterns.md)
