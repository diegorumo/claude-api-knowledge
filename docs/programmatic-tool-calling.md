# Programmatic Tool Calling

> **Last updated:** 2026-06-22  
> **Status:** GA (requires `code_execution_20260120` or later)

Programmatic tool calling lets Claude write Python code that invokes your tools from within a [code execution](./tool-use.md#built-in-tools) sandbox, eliminating per-tool model round-trips. Intermediate tool results are never loaded into Claude's context window — only the final code output is. This reduces both latency and input tokens significantly for multi-tool workflows.

**Benchmark results (Anthropic internal):**
- On a 75-tool project-management benchmark: ~38% fewer billed input tokens, no accuracy change
- On agentic search benchmarks (BrowseComp, DeepSearchQA): +11% performance, 24% fewer input tokens
- Sequential single-call workflows: no meaningful savings (8% higher cost due to container overhead)

## Model Compatibility

Requires `code_execution_20260120` or `code_execution_20260521`:

| Model ID |
|----------|
| `claude-fable-5` |
| `claude-mythos-5` |
| `claude-opus-4-8` |
| `claude-opus-4-7` |
| `claude-opus-4-6` |
| `claude-sonnet-4-6` |
| `claude-opus-4-5-20251101` |
| `claude-sonnet-4-5-20250929` |

Available on: Claude API, Claude Platform on AWS, Microsoft Foundry. **Not available** on Amazon Bedrock or Vertex AI.

Not eligible for Zero Data Retention (ZDR).

## Quick Start

Add `code_execution` to your tools list and set `allowed_callers: ["code_execution_20260120"]` on any tool you want invoked from code:

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": "Query sales data for West, East, and Central regions, then tell me which had highest revenue",
    }],
    tools=[
        {"type": "code_execution_20260120", "name": "code_execution"},
        {
            "name": "query_database",
            "description": "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
            "input_schema": {
                "type": "object",
                "properties": {
                    "sql": {"type": "string", "description": "SQL query to execute"}
                },
                "required": ["sql"],
            },
            "allowed_callers": ["code_execution_20260120"],
        },
    ],
)
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-opus-4-8",
  max_tokens: 4096,
  messages: [{
    role: "user",
    content: "Query sales data for West, East, and Central regions, then tell me which had highest revenue",
  }],
  tools: [
    { type: "code_execution_20260120", name: "code_execution" },
    {
      name: "query_database",
      description: "Execute a SQL query against the sales database. Returns a list of rows as JSON objects.",
      input_schema: {
        type: "object" as const,
        properties: { sql: { type: "string", description: "SQL query to execute" } },
        required: ["sql"],
      },
      allowed_callers: ["code_execution_20260120"],
    },
  ],
});
```

## How It Works

1. Claude writes Python code that calls your tools via `await` (tools are exposed as async functions)
2. The code runs in Anthropic's sandboxed container via code execution
3. When a tool function is called, the container pauses and the API returns a `tool_use` block
4. You provide the tool result — code execution resumes without any model re-sampling
5. Once all code completes, Claude receives the final output and generates its response

The intermediate tool results are **not** added to Claude's context — only the final code execution output is. This is the key token savings mechanism.

## The `allowed_callers` Field

```json
{
  "name": "my_tool",
  "allowed_callers": ["code_execution_20260120"]
}
```

| Value | Meaning |
|-------|---------|
| `["direct"]` | Claude calls this tool directly (default if `allowed_callers` omitted) |
| `["code_execution_20260120"]` | Claude calls this tool only from code |
| `["direct", "code_execution_20260120"]` | Either path allowed |

**Important:** `allowed_callers` is guidance, not a hard security boundary. Claude is strongly steered to follow it, but your client should still handle direct `tool_use` for any tool it defines.

Both `"code_execution_20260120"` and `"code_execution_20260521"` are equivalent in `allowed_callers`. Response blocks always tag the caller as `code_execution_20260120` regardless of which version the request declared.

## The `caller` Field in Responses

Every `tool_use` block includes a `caller` field showing how it was invoked:

**Direct invocation:**
```json
{
  "type": "tool_use",
  "id": "toolu_abc123",
  "name": "query_database",
  "input": { "sql": "SELECT ..." },
  "caller": { "type": "direct" }
}
```

**Programmatic invocation:**
```json
{
  "type": "tool_use",
  "id": "toolu_def456",
  "name": "query_database",
  "input": { "sql": "SELECT ..." },
  "caller": {
    "type": "code_execution_20260120",
    "tool_id": "srvtoolu_abc123"
  }
}
```

`tool_id` references the code execution container that made the call.

## Complete Flow Example

### Step 2 — API response (tool call from code)

```json
{
  "role": "assistant",
  "content": [
    { "type": "text", "text": "I'll query and analyze the results." },
    {
      "type": "server_tool_use",
      "id": "srvtoolu_abc123",
      "name": "code_execution",
      "input": {
        "code": "results = await query_database('<sql>')\ntop = max(results, key=lambda x: x['revenue'])\nprint(top)"
      }
    },
    {
      "type": "tool_use",
      "id": "toolu_def456",
      "name": "query_database",
      "input": { "sql": "<sql>" },
      "caller": { "type": "code_execution_20260120", "tool_id": "srvtoolu_abc123" }
    }
  ],
  "container": { "id": "container_xyz789", "expires_at": "2026-01-20T14:30:00Z" },
  "stop_reason": "tool_use"
}
```

### Step 3 — Provide result (tool_result only, no text)

```python
response2 = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=4096,
    container="container_xyz789",  # reuse the container
    messages=[
        # ... full prior conversation ...
        {
            "role": "user",
            "content": [{
                "type": "tool_result",
                "tool_use_id": "toolu_def456",
                "content": '[{"region": "West", "revenue": 120000}, ...]',
            }],
        },
    ],
    tools=[...],
)
```

**Critical:** When responding to programmatic tool calls, the user message must contain **only** `tool_result` blocks — no text content alongside them.

### Step 5 — Final response

```json
{
  "content": [
    {
      "type": "code_execution_tool_result",
      "tool_use_id": "srvtoolu_abc123",
      "content": {
        "type": "code_execution_result",
        "stdout": "{'region': 'West', 'revenue': 120000}",
        "stderr": "",
        "return_code": 0,
        "content": []
      }
    },
    { "type": "text", "text": "The West region had the highest revenue at $120,000." }
  ],
  "stop_reason": "end_turn"
}
```

## Container Lifecycle

Containers are shared between code execution and programmatic tool calling:

| Property | Value |
|----------|-------|
| Max lifetime | 30 days |
| Idle timeout | 4.5 minutes |
| Reuse | Pass `container` ID to reuse state |
| Watch | `expires_at` field in response |

If the container expires while waiting for a tool result, Claude may treat it as a `TimeoutError` and retry.

## Advanced Patterns

### Batch loop (reduces N round-trips to 1)

```python
# Claude writes this code in the sandbox:
regions = ["West", "East", "Central", "North", "South"]
results = {}
for region in regions:
    data = await query_database(f"SELECT * FROM sales WHERE region='{region}'")
    results[region] = sum(row["revenue"] for row in data)
top = max(results.items(), key=lambda x: x[1])
print(f"Top region: {top[0]} with ${top[1]:,}")
```

### Early termination

```python
for endpoint in ["us-east", "eu-west", "apac"]:
    status = await check_health(endpoint)
    if status == "healthy":
        print(f"Using {endpoint}")
        break
```

### Conditional tool selection

```python
file_info = await get_file_info(path)
if file_info["size"] < 10000:
    content = await read_full_file(path)
else:
    content = await read_file_summary(path)
print(content)
```

## Limitations

- `strict: true` tools cannot be called programmatically
- `disable_parallel_tool_use: true` is incompatible
- `tool_choice` cannot name a tool whose `allowed_callers` omits `"direct"`
- MCP connector tools cannot be called programmatically
- Each programmatic tool call counts against rate limits

## Pricing

Programmatic tool calling uses the same pricing as code execution. Tool results from programmatic invocations do not count toward input/output token usage — only the final code execution result counts.

## When to Use

| Use case | Fit |
|----------|-----|
| Fan-out: check 20+ items in a loop | Strong |
| Large results that can be filtered/aggregated in code | Strong |
| Agentic search with iterative querying | Strong |
| Sequential workflow where each call needs Claude reasoning | Weak |
| Small number of calls with small responses | Weak |
| Requires immediate user feedback between calls | Weak |

## Related

- [Tool Use](./tool-use.md) — fundamentals and tool definition properties
- [Web Search](./web-search.md) — built-in server tools
- [Agent Patterns](./agent-patterns.md) — orchestration and parallelization patterns
