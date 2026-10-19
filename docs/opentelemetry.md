# OpenTelemetry Export (Beta)

> **Last updated:** 2026-10-19  
> **Status:** Beta (launched Oct 14, 2026)  
> **Sources:** platform.claude.com/docs/en/manage-claude/opentelemetry, platform.claude.com/docs/en/manage-claude/opentelemetry/sdk, platform.claude.com/docs/en/manage-claude/opentelemetry/api, platform.claude.com/docs/en/release-notes/overview (Oct 14, 2026)

**Summary:** Trace Messages API requests, tool uses and Managed Agents sessions in your own observability backend. Two sources of spans: the Claude SDK (client side, set up in code) and the Claude API (server side, set up as a destination in the Console). Use both to see SDK and API spans in one trace. Spans use the [GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai).

> **Beta warning (from the SDK page):** SDK options and the span schema can change in ways that break code, queries or dashboards, for example a renamed span, attribute, option or environment variable. Pin your SDK version.

## SDK tracing vs API tracing

| What you get                                   | SDK tracing           | API tracing                                  |
| ---------------------------------------------- | --------------------- | -------------------------------------------- |
| Messages API requests                          | ✅                    | ✅                                           |
| Sessions API requests                          | ✅                    | ✅                                           |
| Token usage                                    | ✅                    | ✅                                           |
| Each model run within a request                | —                     | ✅                                           |
| Managed Agents turns                           | —                     | ✅                                           |
| Server tools, run by the API                   | —                     | ✅                                           |
| Client tools, run by the tool runner           | ✅                    | —                                            |
| Tools in a self-hosted sandbox                 | ✅                    | —                                            |
| Other API requests, such as uploading files    | ✅                    | —                                            |
| Setup                                          | Code                  | Console (or Admin API)                       |
| Required permission                            | None                  | Admin                                        |
| Coverage                                       | Clients using the SDK | Any client                                   |
| Cloud platform support                         | Any                   | Claude Platform, Claude Platform on AWS      |

SDK spans are emitted by the Python, TypeScript, C#, Go, Java, PHP and Ruby SDKs.

## Content modes

- **Metadata only** (`metadata_only`, default): identifiers, timing, status, token counts, and names/descriptions you chose (agents, tools, MCP servers).
- **Full content** (`content`): also model prompts and responses.

Organizations with a HIPAA, FERPA or K-12 data-handling restriction get metadata only from API destinations regardless of the setting (SDK spans aren't affected). Full content works with zero data retention for Messages API requests (held in memory until delivered). Managed Agents isn't ZDR-eligible.

**Content limit:** 0.75 MB per span by default. Over the limit, content is shortened and `anthropic.content.truncated` is `true`. A full 1M-token context needs about 4 MB. Change it with `traces.max_content_bytes` in the SDK, or **Maximum content size in MB** in the Console.

## SDK tracing

Install the latest SDK and register an OpenTelemetry tracer provider before the first call. The SDK uses the tracer provider your application registers.

```bash
pip install --upgrade anthropic opentelemetry-sdk
```

```python
from anthropic import Anthropic
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor

# Register a tracer provider that prints each span to the console.
provider = TracerProvider()
provider.add_span_processor(SimpleSpanProcessor(ConsoleSpanExporter()))
trace.set_tracer_provider(provider)

client = Anthropic()

message = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
)
print(message.content)
```

```bash
npm install @anthropic-ai/sdk@latest @opentelemetry/sdk-trace-node
```

```typescript
import Anthropic from "@anthropic-ai/sdk";
import {
  ConsoleSpanExporter,
  NodeTracerProvider,
  SimpleSpanProcessor
} from "@opentelemetry/sdk-trace-node";

// Register a tracer provider that prints each span to the console.
const provider = new NodeTracerProvider({
  spanProcessors: [new SimpleSpanProcessor(new ConsoleSpanExporter())]
});
// register() also installs the context manager, which lets the SDK spans nest under your spans.
provider.register();

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude" }]
});
console.log(message.content);
```

Swap the console exporter for an OTLP exporter to send spans to your platform. Java needs `com.anthropic:anthropic-java` 2.73.0 or later.

### SDK options

Passing `open_telemetry` (TypeScript: `openTelemetry`) to the client overrides all `ANTHROPIC_OPEN_TELEMETRY*` environment variables as a whole; nested keys you leave out take their defaults.

| Option (Python / TypeScript)                            | Environment variable                                 | Use it to                                                         | Default         |
| ------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------- | --------------- |
| `traces.content_mode` / `traces.contentMode`            | `ANTHROPIC_OPEN_TELEMETRY_TRACES_CONTENT_MODE`       | Include prompts and responses (`content`)                         | `metadata_only` |
| `traces.max_content_bytes` / `traces.maxContentBytes`   | `ANTHROPIC_OPEN_TELEMETRY_TRACES_MAX_CONTENT_BYTES`  | Change the per-span content limit                                 | 0.75 MB         |
| `conversation_id` / `conversationId`                    | —                                                    | Link calls into one conversation (`gen_ai.conversation.id`); exported in both modes, so no personal data | None |
| `tracer_provider` / `tracerProvider`                    | —                                                    | Choose the tracer provider                                        | The registered provider |
| `propagation`                                           | `ANTHROPIC_OPEN_TELEMETRY_PROPAGATION=false`         | Stop sending trace context and baggage headers                    | On              |
| `open_telemetry` / `openTelemetry`                      | `ANTHROPIC_OPEN_TELEMETRY=false`                     | Turn spans off for one client                                     | On              |

```python
client = Anthropic(
    open_telemetry={"tracer_provider": provider, "traces": {"content_mode": "content"}}
)

quiet_client = Anthropic(open_telemetry=False)

# with_options() replaces the whole option, so start from client.open_telemetry.
message = client.with_options(
    open_telemetry={**client.open_telemetry, "conversation_id": "conv_42"}
).messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude"}],
)
```

```typescript
const client = new Anthropic({
  openTelemetry: { tracerProvider: provider, traces: { contentMode: "content" } }
});

const quietClient = new Anthropic({ openTelemetry: false });

const message = await client.messages.create(
  {
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  },
  { openTelemetry: { conversationId: "conv_42" } }
);
```

## API tracing (destinations)

Needs the **admin** role in the Console organization (developers can view), an **OTLP/HTTP endpoint** accepting protobuf over HTTPS on port 443, and credentials for it.

1. Console → [OpenTelemetry settings](https://platform.claude.com/settings/telemetry-destinations) → **Add destination**.
2. Set **Provider** (a listed partner or **Custom**), **OTLP endpoint URL** (base URL with a hostname; don't add `/v1/traces`), **Request headers** (credentials go only in header values), **Content mode**, and **Workspaces** (**All**, **Selected** or **All except**).
3. Save, then **Test** from the **More actions** menu and look for the `anthropic.test` span.
4. Send a request in a covered workspace and filter on `service.name` = `claude-platform`.

Or create the destination with the Admin API (beta header `telemetry-destinations-2026-08-11`), using an Admin API key:

```python
client = anthropic.Anthropic()

destination = client.beta.organization.telemetry_destinations.create(
    endpoint_url="https://gateway.example.com/otlp",
    headers={"authorization": "Bearer YOUR_OBSERVABILITY_API_KEY"},
    traces={"content_mode": "metadata_only"},
    workspaces={"type": "all"},
)
print(destination.id)
```

```typescript
const client = new Anthropic();

const destination = await client.beta.organization.telemetryDestinations.create({
  endpoint_url: "https://gateway.example.com/otlp",
  headers: { authorization: "Bearer YOUR_OBSERVABILITY_API_KEY" },
  traces: { content_mode: "metadata_only" },
  workspaces: { type: "all" }
});
console.log(destination.id);
```

On Claude Platform on AWS, send these requests to `https://aws-external-anthropic.{region}.api.aws` signed with AWS credentials; your IAM policy decides who can manage destinations.

### Connecting API spans to your traces

The API reads the W3C `traceparent` header. With SDK tracing set up, call the SDK inside your own span and the API span nests under the SDK span. If the sampled flag in `traceparent` is off, the API emits no spans for that request.

### Delivery

- **At least once** (dedupe on trace ID + span ID) and **best effort** (lost spans aren't resent).
- gzip on first attempt; on a `400`/`415` it resends uncompressed.
- Retries timeouts, connection/TLS/DNS failures, `408`, `429` and `5xx`. 5 s TLS handshake, 5 s to response headers, 10 s total.
- `User-Agent: Claude-Platform`.
- A destination is **disabled** on a `3xx` redirect, a hostname resolving to a non-public IP, or 24 hours of failed deliveries. Fix the cause, then **Enable** it.

### Limits

| Limit                                                       | Value            |
| ----------------------------------------------------------- | ---------------- |
| Destinations for each organization, including disabled ones | 10               |
| Workspaces that you select for a destination                | 1,000            |
| Request headers for each destination                        | 20               |
| Length of a header name                                     | 128 bytes        |
| Length of a header value                                    | 4,096 bytes      |
| Length of all header names and values together              | 16,384 bytes     |
| Length of an endpoint URL                                   | 2,048 characters |

## Counting tokens from spans

Sum one source per request. When a request has both, sum the API spans (SDK span is `CLIENT`, API span is `SERVER`, both named `anthropic.messages.create`).

- **SDK spans:** sum `anthropic.messages.create`; add `anthropic.usage.compaction.input_tokens` / `output_tokens` when there were compactions.
- **API spans, Messages API:** sum either `anthropic.messages.create` or `anthropic.model_request`, not both; add `anthropic.compaction` spans.
- **API spans, Managed Agents:** sum `anthropic.model_request` and `anthropic.session.outcome_evaluation`.

`gen_ai.usage.input_tokens` already includes cache reads and writes. Spans aren't a complete usage record; use the [Usage API](https://platform.claude.com/docs/en/manage-claude/usage-cost-api) for that.

## Grouping Managed Agents spans

- One thread: group by `gen_ai.conversation.id` (the session thread ID).
- Whole session, all threads: group by `anthropic.session.id`, across traces.

## Related

- [Managed Agents](./managed-agents.md)
- [Tool Use](./tool-use.md)
- [OpenTelemetry span reference (official)](https://platform.claude.com/docs/en/api/opentelemetry/spans)
- For Claude Code, the Agent SDK or Cowork, see their own monitoring docs (linked from the [official overview](https://platform.claude.com/docs/en/manage-claude/opentelemetry)).
