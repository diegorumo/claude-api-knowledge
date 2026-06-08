# Tool Use / Function Calling

> **Last updated:** 2026-06-08

## Overview

Tool use lets Claude call functions you define. Claude decides when and which tool to use based on the user's request. The pattern is:

1. Send request with tool definitions
2. Claude responds with `stop_reason: "tool_use"` and a `tool_use` content block
3. Execute the tool locally
4. Send results back in a `tool_result` message
5. Claude generates the final response

## Tool Definition

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location. Returns temperature in Fahrenheit.",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name, e.g. 'San Francisco, CA'"
                },
                "unit": {
                    "type": "string",
                    "enum": ["fahrenheit", "celsius"],
                    "description": "Temperature unit"
                }
            },
            "required": ["location"]
        }
    }
]
```

## Complete Python Example

```python
from anthropic import Anthropic
from anthropic.types import ToolParam, MessageParam

client = Anthropic()

user_message: MessageParam = {
    "role": "user",
    "content": "What is the weather in San Francisco?",
}

tools = [
    {
        "name": "get_weather",
        "description": "Get the weather for a specific location",
        "input_schema": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"],
        },
    }
]

# Step 1: Send request
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[user_message],
    tools=tools,
)

# Step 2: Check if tool use was requested
assert response.stop_reason == "tool_use"
tool = next(c for c in response.content if c.type == "tool_use")
print(f"Tool: {tool.name}, Input: {tool.input}")

# Step 3: Execute tool (your logic here)
def get_weather(location):
    return f"72°F and sunny in {location}"

tool_result = get_weather(tool.input["location"])

# Step 4: Send result back
final = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        user_message,
        {"role": response.role, "content": response.content},
        {
            "role": "user",
            "content": [
                {
                    "type": "tool_result",
                    "tool_use_id": tool.id,
                    "content": [{"type": "text", "text": tool_result}],
                }
            ],
        },
    ],
    tools=tools,
)
print(final.content[0].text)
```

## Complete TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const tools = [
  {
    name: 'get_weather',
    description: 'Get the weather for a specific location',
    input_schema: {
      type: 'object' as const,
      properties: { location: { type: 'string' } },
      required: ['location'],
    },
  },
];

const userMessage = { role: 'user' as const, content: 'What is the weather in SF?' };

// Step 1: Initial request
const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [userMessage],
  tools,
});

// Step 2: Find tool use block
const toolUse = response.content.find((b) => b.type === 'tool_use');
if (!toolUse || toolUse.type !== 'tool_use') throw new Error('No tool use');

// Step 3: Execute tool
const toolResult = `72°F and sunny in ${(toolUse.input as any).location}`;

// Step 4: Send result back
const final = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [
    userMessage,
    { role: response.role, content: response.content },
    {
      role: 'user',
      content: [
        {
          type: 'tool_result',
          tool_use_id: toolUse.id,
          content: [{ type: 'text', text: toolResult }],
        },
      ],
    },
  ],
  tools,
});
console.log(final.content[0].text);
```

## Agentic Loop (Multiple Tools)

```python
def run_agent(client, tools, tool_functions, initial_message):
    messages = [{"role": "user", "content": initial_message}]
    
    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages,
        )
        
        if response.stop_reason == "end_turn":
            return response.content[0].text
        
        if response.stop_reason == "tool_use":
            messages.append({"role": response.role, "content": response.content})
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    fn = tool_functions.get(block.name)
                    result = fn(**block.input) if fn else "Tool not found"
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result),
                    })
            
            messages.append({"role": "user", "content": tool_results})
```

## Tool Choice

Control which tool Claude uses:

```python
# Auto (default) - Claude decides
tool_choice = {"type": "auto"}

# Force a specific tool
tool_choice = {"type": "tool", "name": "get_weather"}

# Force any tool (but Claude chooses which)
tool_choice = {"type": "any"}

# No tools (even if defined)
tool_choice = {"type": "none"}

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    tool_choice=tool_choice,
    messages=messages,
)
```

## Tool Result Error

```python
tool_results.append({
    "type": "tool_result",
    "tool_use_id": block.id,
    "content": "Error: location not found",
    "is_error": True,
})
```

## Built-in Tools

The API includes server-side tools that don't require local execution. Use the **latest** type version for each tool family:

```python
# Web search (Claude searches the web)
tools = [{"type": "web_search_20260209", "name": "web_search"}]  # latest; also: web_search_20250305

# Web fetch (Claude fetches a URL)
tools = [{"type": "web_fetch_20260309", "name": "web_fetch"}]  # latest; also: web_fetch_20260209

# Code execution (runs code in a sandbox)
tools = [{"type": "code_execution_20260120", "name": "code_execution"}]  # latest; also: 20250522, 20250825

# Memory (persistent agent memory)
tools = [{"type": "memory_20250818", "name": "memory"}]

# Text editor (file editing operations)
tools = [{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}]

# BM25 search over provided documents
tools = [{"type": "search_bm25_20251119", "name": "bm25_search"}]

# Regex search over provided documents
tools = [{"type": "search_regex_20251119", "name": "regex_search"}]

# Bash execution
tools = [{"type": "bash_20250124", "name": "bash"}]
```

**Version history for key tools:**

| Tool | Versions (newest first) |
|------|------------------------|
| Web Search | `web_search_20260209`, `web_search_20250305` |
| Web Fetch | `web_fetch_20260309`, `web_fetch_20260209` |
| Code Execution | `code_execution_20260120`, `code_execution_20250825`, `code_execution_20250522` |
| Text Editor | `text_editor_20250728`, `text_editor_20250429`, `text_editor_20250124` |

## Tool Use Response Content Block

```json
{
  "type": "tool_use",
  "id": "toolu_01A09q90qw90lq917835lq9",
  "name": "get_weather",
  "input": {"location": "San Francisco, CA"}
}
```

## Tool Result Content Block

```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
  "content": [{"type": "text", "text": "72°F and sunny"}]
}
```

## Best Practices

- Write clear, specific descriptions — they directly influence Claude's tool selection
- Use `required` in `input_schema` to make required parameters explicit
- Return errors via `is_error: true` rather than raising exceptions — Claude will handle them gracefully
- Cap agentic loops (e.g. max 10 iterations) to prevent runaway execution
- Log all tool calls for debugging

## Zod Tool Helpers (TypeScript)

```typescript
import { betaZodTool } from '@anthropic-ai/sdk/helpers/beta/zod';
import { z } from 'zod';

const weatherTool = betaZodTool({
  name: 'get_weather',
  description: 'Get current weather',
  inputSchema: z.object({ location: z.string() }),
  run: ({ location }) => `72°F and sunny in ${location}`,
});

const finalMessage = await client.beta.messages.toolRunner({
  model: 'claude-sonnet-4-6',
  tools: [weatherTool],
  messages: [{ role: 'user', content: 'What is the weather?' }],
  max_tokens: 1024,
});
```

## Related

- [Messages API](./messages-api.md)
- [Streaming](./streaming.md)
- [Web Search](./web-search.md)
- [Agent Patterns](./agent-patterns.md)
- [MCP Integration](./mcp.md)
