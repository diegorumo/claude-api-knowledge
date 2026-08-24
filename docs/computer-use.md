# Computer Use & Browser Use

> **Last updated:** 2026-08-24  
> **Computer use toolset status:** GA as of 2026-08-19 (`computer_toolset_20260801`, no beta header)  
> **Browser use toolset status:** GA as of 2026-08-19 (`browser_toolset_20260801`, no beta header)  
> **Available on:** Claude Fable 5, Claude Mythos 5, Claude Opus 5, Claude Sonnet 5, Claude Opus 4.8

## Overview

Two toolsets let Claude control graphical interfaces:

- **Computer use** (`computer_toolset_20260801`) — full desktop control: screenshots, mouse, keyboard, scroll; operates at the OS level in your sandbox
- **Browser use** (`browser_toolset_20260801`) — browser viewport automation: accessibility tree, form filling, tab management, downloads; operates at the browser level rather than via screenshots

Both are now GA and require no beta header.

**Important:** Always run these in a sandboxed container/VM isolated from sensitive systems.

---

## Computer Use Toolset (GA)

### `computer_toolset_20260801` vs. older beta versions

| Feature | `computer_20241022` (old beta) | `computer_toolset_20260801` (GA) |
|---------|-------------------------------|----------------------------------|
| Beta header required | Yes (`computer-use-2024-10-22`) | No |
| Batch actions | No (one action per turn) | Yes (multiple actions per turn) |
| Zoom | Manual | Enabled by default |
| Per-member config | No | Yes (via `configs`) |

The old beta versions remain available; see [Migrate from `computer_20251124`](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124) for upgrading.

### Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

# GA: no beta header or betas= needed
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    tools=[
        {
            "type": "computer_toolset_20260801",
            "name": "computer",
            "display_width_px": 1920,
            "display_height_px": 1080,
            # zoom enabled by default
        }
    ],
    messages=[{"role": "user", "content": "Open the terminal and list files in home directory."}],
)
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

// GA: no beta header or betas= needed
const response = await client.messages.create({
  model: 'claude-opus-5',
  max_tokens: 4096,
  tools: [
    {
      type: 'computer_toolset_20260801',
      name: 'computer',
      display_width_px: 1920,
      display_height_px: 1080,
    },
  ],
  messages: [{ role: 'user', content: 'Take a screenshot.' }],
});
```

### Agentic Computer Use Loop

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
    client = anthropic.Anthropic()
    
    for step in range(max_steps):
        screenshot_b64 = take_screenshot()
        
        if not messages:
            messages = [{
                "role": "user",
                "content": [
                    {"type": "text", "text": f"Complete this task: {task}"},
                    {"type": "tool_result", "tool_use_id": "initial",
                     "content": [{"type": "image", "source": {"type": "base64",
                                  "media_type": "image/png", "data": screenshot_b64}}]},
                ],
            }]
        
        response = client.messages.create(
            model="claude-opus-5",
            max_tokens=4096,
            tools=[{"type": "computer_toolset_20260801", "name": "computer",
                    "display_width_px": 1280, "display_height_px": 800}],
            messages=messages,
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
                    "content": [{"type": "image", "source": {"type": "base64",
                                 "media_type": "image/png",
                                 "data": result or take_screenshot()}}],
                })
        
        if tool_results:
            messages.append({"role": "user", "content": tool_results})
```

### Computer Action Types

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

---

## Browser Use Toolset (GA)

`browser_toolset_20260801` operates at the browser viewport level rather than the full desktop. Claude reads the page's accessibility tree and element structure rather than relying solely on screenshots.

### Key Capabilities

- **Accessibility tree reading** — understands page structure without screenshots
- **Element interaction** — click by element reference, not pixel coordinate
- **Form filling** — direct value injection into inputs
- **Tab management** — open, close, switch tabs
- **Download reporting** — knows when files are downloaded
- **Opt-in file upload** — upload files to web forms

### Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

# GA: no beta header needed
response = client.messages.create(
    model="claude-opus-5",
    max_tokens=4096,
    tools=[
        {
            "type": "browser_toolset_20260801",
            "name": "browser",
        }
    ],
    messages=[{
        "role": "user",
        "content": "Go to example.com and fill in the contact form with my name 'Alice'."
    }],
)
```

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const response = await client.messages.create({
  model: 'claude-opus-5',
  max_tokens: 4096,
  tools: [
    {
      type: 'browser_toolset_20260801',
      name: 'browser',
    },
  ],
  messages: [{ role: 'user', content: 'Navigate to example.com and click the "Sign In" link.' }],
});
```

### When to Use Computer Use vs Browser Use

| Scenario | Recommendation |
|----------|---------------|
| Full desktop automation (file system, desktop apps) | Computer use |
| Web-only tasks where you control the browser | Browser use |
| Form filling and web interaction | Browser use (more reliable) |
| Tasks requiring visual inspection of non-web content | Computer use |

---

## Security Considerations

1. Run in an isolated container/VM
2. Limit network access from the sandbox
3. Don't pass credentials or sensitive data via the GUI
4. Implement action confirmation for destructive operations
5. Set time limits on agent runs

## Gotchas

- **GA as of 2026-08-19**: no beta header required for `computer_toolset_20260801` or `browser_toolset_20260801`
- Old beta versions (`computer_20241022`, `computer_20251124`) still work with their beta headers
- High-resolution displays consume more tokens per screenshot — use 1280×800 for efficiency
- Tool type identifiers include version dates — don't omit the date
- Screenshot quality affects Claude's ability to read text — use lossless PNG
- Batch actions in `computer_toolset_20260801` can reduce round-trips for simple sequences

## Related

- [Tool Use](./tool-use.md)
- [Agent Patterns](./agent-patterns.md)
- [Managed Agents](./managed-agents.md)
