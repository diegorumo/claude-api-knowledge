# Computer Use

> **Last updated:** 2026-05-30  
> **Status:** Beta (`anthropic-beta: computer-use-2024-10-22`)  
> **Available on:** Claude Sonnet 4.6, Claude Opus 4.8

## Overview

Computer use allows Claude to interact with a computer interface — taking screenshots, moving the mouse, clicking, and typing — to complete tasks autonomously.

**Important:** Always run in a sandboxed container/VM isolated from sensitive systems.

## Computer Use Tools

```python
tools = [
    {
        "type": "computer_20241022",
        "name": "computer",
        "display_width_px": 1920,
        "display_height_px": 1080,
        "display_number": 1,
    },
    {"type": "text_editor_20241022", "name": "str_replace_editor"},
    {"type": "bash_20241022", "name": "bash"},
]
```

## Basic Computer Use Loop

```python
import base64
import subprocess
from pathlib import Path

def take_screenshot() -> str:
    subprocess.run(["scrot", "/tmp/screenshot.png"], check=True)
    data = Path("/tmp/screenshot.png").read_bytes()
    return base64.standard_b64encode(data).decode("utf-8")

def execute_computer_action(action: dict):
    action_type = action["type"]
    if action_type == "screenshot":
        return take_screenshot()
    elif action_type == "left_click":
        subprocess.run(["xdotool", "click", "1",
                       str(action["coordinate"][0]), str(action["coordinate"][1])])
    elif action_type == "type":
        subprocess.run(["xdotool", "type", "--clearmodifiers", action["text"]])
    elif action_type == "key":
        subprocess.run(["xdotool", "key", "--clearmodifiers", action["key"]])

def run_computer_use(task: str, max_steps: int = 20):
    messages = []
    
    for step in range(max_steps):
        screenshot_b64 = take_screenshot()
        
        if not messages:
            messages = [{
                "role": "user",
                "content": [
                    {"type": "text", "text": f"Complete this task: {task}"},
                    {"type": "tool_result", "tool_use_id": "initial",
                     "content": [{"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": screenshot_b64}}]},
                ],
            }]
        
        response = client.beta.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages,
            betas=["computer-use-2024-10-22"],
        )
        
        messages.append({"role": "assistant", "content": response.content})
        
        if response.stop_reason == "end_turn":
            return
        
        tool_results = []
        for block in response.content:
            if block.type == "tool_use" and block.name == "computer":
                result = execute_computer_action(block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": [{"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": result or take_screenshot()}}],
                })
        
        if tool_results:
            messages.append({"role": "user", "content": tool_results})
```

## Computer Action Types

| Action | Description | Parameters |
|--------|-------------|----------|
| `screenshot` | Take a screenshot | (none) |
| `left_click` | Left click | `coordinate: [x, y]` |
| `right_click` | Right click | `coordinate: [x, y]` |
| `double_click` | Double click | `coordinate: [x, y]` |
| `type` | Type text | `text: string` |
| `key` | Press a key | `key: string` |
| `scroll` | Scroll | `coordinate`, `direction`, `amount` |
| `mouse_move` | Move mouse | `coordinate: [x, y]` |

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const response = await client.beta.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 4096,
  tools: [
    {
      type: 'computer_20241022',
      name: 'computer',
      display_width_px: 1920,
      display_height_px: 1080,
    },
  ],
  messages: [{ role: 'user', content: 'Open the terminal and list files in home directory.' }],
  betas: ['computer-use-2024-10-22'],
});
```

## Security Considerations

1. Run Claude in an isolated container/VM
2. Limit network access from the sandbox
3. Don't pass credentials or sensitive data via the GUI
4. Implement action confirmation for destructive operations
5. Set time limits on agent runs

## Gotchas

- Requires `computer-use-2024-10-22` beta flag
- High-resolution displays consume more tokens per screenshot — use 1280×800 for efficiency
- Tool type identifiers include version dates (e.g., `computer_20241022`) — don't omit the date
- Screenshot quality affects Claude's ability to read text — use lossless PNG

## Related

- [Tool Use](./tool-use.md)
- [Agent Patterns](./agent-patterns.md)
- [Managed Agents](./managed-agents.md)
