# Managed Agents (Beta)

> **Last updated:** 2026-06-15  
> **Status:** Beta — active development  
> **SDK changelog:** v0.100.0+ (May 2026), v0.109.0 (June 2026)

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

## Gotchas

- Managed Agents is under active development — check SDK changelogs for new features
- Sessions have inactivity timeouts — implement keep-alive or reconnection logic
- Custom tool results must be returned promptly or the session may timeout
- v0.103.0+: self-hosted sandboxes supported
- v0.109.0+: deployments with environment variable credentials supported

## Related

- [Tool Use](./tool-use.md)
- [MCP Integration](./mcp.md)
- [Files API](./files-api.md)
- [Agent Patterns](./agent-patterns.md)
