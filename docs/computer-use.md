# Computer Use & Browser Use

> **Last updated:** 2026-10-10  
> **Computer use toolset status:** GA as of 2026-08-19 (`computer_toolset_20260801`, no beta header)  
> **Browser use toolset status:** GA as of 2026-08-19 (`browser_toolset_20260801`, no beta header)  
> **Available on:** Claude Fable 5, Claude Mythos 5, Claude Opus 5, Claude Sonnet 5, Claude Opus 4.8  
> **Claude Haiku 5.5 (Oct 7, 2026):** supports `computer_toolset_20260801` and `browser_toolset_20260801` on the Claude API and Google Cloud; on Amazon Bedrock use `computer_20251124` (beta `computer-use-2025-11-24`). `computer_20250124` returns 400 on Haiku 5.5 on every platform.

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

## SDK Toolset Classes (Beta, Oct 7, 2026)

The Python and TypeScript SDKs include classes for the browser use tool and the computer use tool. You subclass one and write one method per member tool (such as `navigate` or `left_click`) against your own browser or desktop automation. The SDK runs the tool loop, your URL and file policies (browser only) and your approval callback. Source: [Browser and computer use with the SDK toolsets](https://platform.claude.com/docs/en/agents-and-tools/tool-use/browser-use-sdk).

| Class                                  | Python import                | TypeScript import                         |
| -------------------------------------- | ---------------------------- | ----------------------------------------- |
| `BetaAbstractBrowserToolset20260801`   | `anthropic.tools.browser`    | `@anthropic-ai/sdk/helpers/beta/toolsets` |
| `BetaAbstractComputerToolset20260801`  | `anthropic.tools.computer`   | (see the TypeScript computer toolset guide) |

- The SDK ships **no** browser, desktop, ready-made driver or URL policy. Minimal CDP (browser) and VNC (computer) examples are in the `claude-quickstarts` repo; they aren't production code. Browser Use, Browserbase, Daytona and E2B publish their own integrations.
- Pass the driver instance itself as the `tools` entry of the tool runner. Members you don't implement are sent as disabled. The runner never closes the toolset; close it yourself.
- Constructor options (both SDKs, can't change after construction): `configs`, `confirm`, `url_policy` / `urlPolicy`, `file_policy` / `filePolicy`, `tool_configs` / `toolConfigs`, and the `_browser_state` method / `browserState` option (required state report). The computer class takes only `configs`, `confirm` and `tool_configs`.
- Computer class: if you implement `type`, `key` or `hold_key`, the constructor raises a configuration error unless you pass `confirm` or disable those tools with `configs`. The SDK doesn't resize computer screenshots; the API rejects images over the model's limits.
- Limitations: the URL policy sees only `navigate` calls (not redirects, link clicks or subrequests); approvals use the last state report; calls on one toolset run one at a time.
- Before running against anything but a throwaway browser: write a URL policy, block private ranges and `169.254.169.254` with container egress rules, and use a browser profile that isn't signed in to accounts you wouldn't hand to Claude.

Trimmed from the official quick start (`backend` is your own wrapper around e.g. Playwright):

```python
from anthropic import Anthropic
from anthropic.tools import ToolError
from anthropic.tools.browser import (
    BetaAbstractBrowserToolset20260801,
    BetaBrowserNavigateResult,
    BetaBrowserState,
    BetaToolsetCallContext,
    BetaURLContext,
)
from anthropic.types.beta import BetaBrowserNavigateInput, BetaBrowserStateTabEntryParam


class MyBrowser(BetaAbstractBrowserToolset20260801):
    def __init__(self, backend, **options):
        super().__init__(**options)
        self.backend = backend

    def _browser_state(self, context: BetaToolsetCallContext) -> BetaBrowserState:
        return BetaBrowserState(
            tabs=[
                BetaBrowserStateTabEntryParam(
                    tab_id=tab.id, title=tab.title, url=tab.url,
                    active=tab.id == self.backend.active,
                )
                for tab in self.backend.tabs()
            ],
            state_changes=self.backend.drain_changes(),
        )

    def navigate(
        self, context: BetaToolsetCallContext, input: BetaBrowserNavigateInput
    ) -> BetaBrowserNavigateResult:
        page = self.backend.goto(input.url, input.tab_id)
        return BetaBrowserNavigateResult(url=page.url, status=page.status, title=page.title)


def url_policy(context: BetaURLContext, url: str) -> None:
    if not is_allowed(url):  # your own allowlist check
        raise ToolError(f"blocked: {url} is not on an allowed host")


client = Anthropic()
with MyBrowser(backend, url_policy=url_policy) as browser:
    runner = client.beta.messages.tool_runner(
        model="claude-opus-5-5",
        max_tokens=1024,
        tools=[browser],
        messages=[{"role": "user", "content": "Open example.com and tell me the page heading."}],
        stream=True,
        run_tools_eagerly=True,  # so a call can start before the response ends
    )
    for stream in runner:
        print(stream.get_final_message())
```

```typescript
import Anthropic from "@anthropic-ai/sdk";
import {
  BetaAbstractBrowserToolset20260801,
  type BetaBrowserNavigateResult,
  type BetaBrowserToolsetOptions,
  type BetaToolsetCallContext,
  type BetaURLContext,
  ToolError,
} from "@anthropic-ai/sdk/helpers/beta/toolsets";
import type { BetaBrowserNavigateInput } from "@anthropic-ai/sdk/resources/beta";

class MyBrowser extends BetaAbstractBrowserToolset20260801 {
  constructor(private backend: Backend, options: Omit<BetaBrowserToolsetOptions, "browserState"> = {}) {
    super({
      ...options,
      browserState: () => ({
        tabs: backend.tabs().map((tab) => ({
          tab_id: tab.id, title: tab.title, url: tab.url, active: tab.id === backend.active,
        })),
        state_changes: backend.drainChanges(),
      }),
    });
  }

  protected override async navigate(
    ctx: BetaToolsetCallContext,
    input: BetaBrowserNavigateInput,
  ): Promise<BetaBrowserNavigateResult> {
    const page = await this.backend.goto(input.url, input.tab_id);
    return { url: page.url, status: page.status, title: page.title };
  }
}

function urlPolicy(ctx: BetaURLContext, url: string): void {
  if (!isAllowed(url)) throw new ToolError(`blocked: ${url} is not on an allowed host`);
}

const client = new Anthropic();
const browser = new MyBrowser(backend, { urlPolicy });
try {
  const runner = client.beta.messages.toolRunner({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    tools: [browser],
    messages: [{ role: "user", content: "Open example.com and tell me the page heading." }],
    stream: true,
    runToolsEagerly: true,
  });
  for await (const stream of runner) console.log(await stream.finalMessage());
} finally {
  await browser.close();
}
```

The SDK versions that first shipped these classes weren't confirmed this run (SDK changelogs not fetched).

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
