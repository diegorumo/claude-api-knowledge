# Fast Mode

> **Last updated:** 2026-10-19  
> **Sources:** platform.claude.com/docs/en/build-with-claude/fast-mode, platform.claude.com/docs/en/about-claude/pricing (Fast mode pricing), platform.claude.com/docs/en/release-notes/overview (Oct 15, 2026)

**Summary:** Up to 2.5x higher output tokens per second from the same model at premium pricing. Opt in with `speed: "fast"` on a Messages API request.

## Status (Oct 15, 2026)

| Model                               | Status                    | Beta header                                   | Platforms                                                                                       |
| ----------------------------------- | ------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Claude Opus 5.5 (`claude-opus-5-5`) | Out of research preview   | Not needed (`fast-mode-2026-02-01` still accepted) | Claude API (including Managed Agents); Claude in Microsoft Foundry, deployments hosted on Azure |
| Claude Opus 5 (`claude-opus-5`)     | Research preview          | `fast-mode-2026-02-01` required               | Claude API (including Managed Agents)                                                           |
| Claude Opus 4.8 (`claude-opus-4-8`) | Research preview          | `fast-mode-2026-02-01` required               | Claude API (including Managed Agents)                                                           |

- **Opus 5.5:** no access request needed. It has a default fast mode rate limit in the Start, Build and Scale usage tiers. Organizations in the Evaluation tier get it once they move to Start, Build or Scale.
- **Opus 5 / Opus 4.8:** request access through your account manager or the [waitlist](https://claude.com/fast-mode), and keep sending the beta header.
- **Not available:** Claude Platform on AWS, Foundry deployments hosted on Anthropic, other partner-operated cloud platforms, the Batch API, and Priority Tier commitments.

## How it works

Same model weights and behavior, faster inference configuration. The gain is in output tokens per second (OTPS), not time to first token (TTFT), so it shows most when streaming.

## Basic usage

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=4096,
    speed="fast",
    messages=[
        {"role": "user", "content": "Refactor this module to use dependency injection"}
    ],
)

print(response.usage.speed)  # "fast" or "standard"
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 4096,
  speed: "fast",
  messages: [
    { role: "user", content: "Refactor this module to use dependency injection" }
  ]
});

console.log(response.usage.speed); // "fast" or "standard"
```

For Opus 5 and Opus 4.8, add `anthropic-beta: fast-mode-2026-02-01`.

## Pricing

Applies across the full context window, including requests over 200k input tokens.

| Model                           | Input          | Output         |
| ------------------------------- | -------------- | -------------- |
| Claude Opus 5.5                 | $8 / MTok      | $40 / MTok     |
| Claude Opus 5 / Claude Opus 4.8 | $10 / MTok     | $50 / MTok     |

Prompt caching multipliers and data residency multipliers apply on top of fast mode pricing.

## Rate limits

Fast mode has its own rate limit, separate from standard Opus limits. Over the limit, the API returns `429` with a `retry-after` header. Response headers:

| Header                                   | Description                                       |
| ---------------------------------------- | ------------------------------------------------- |
| `anthropic-fast-input-tokens-limit`      | Maximum fast mode input tokens per minute         |
| `anthropic-fast-input-tokens-remaining`  | Remaining fast mode input tokens                  |
| `anthropic-fast-input-tokens-reset`      | Time when the fast mode input token limit resets  |
| `anthropic-fast-output-tokens-limit`     | Maximum fast mode output tokens per minute        |
| `anthropic-fast-output-tokens-remaining` | Remaining fast mode output tokens                 |
| `anthropic-fast-output-tokens-reset`     | Time when the fast mode output token limit resets |

## Checking which speed was used

`usage.speed` is `"fast"` or `"standard"`. `speed: "fast"` on an unsupported model returns an error, and so does running out of fast mode rate limit or capacity (`429` or `529`). Exception: Claude Opus 4.6 with `speed: "fast"` and the beta header succeeds but runs at standard speed, and `usage.speed` says `"standard"`.

## Retries and falling back to standard speed

The SDKs retry fast mode `429`s up to 2 times by default, waiting for `retry-after`. To fall back to standard speed instead, set `max_retries=0` on the fast request, catch the rate limit error and resend without `speed`. Switching speed is a prompt cache miss: fast and standard requests don't share cached prefixes.

```python
import anthropic

client = anthropic.Anthropic()


def create_message_with_fast_fallback(max_retries=0, max_attempts=3, **params):
    try:
        return client.with_options(max_retries=max_retries).messages.create(**params)
    except anthropic.RateLimitError:
        if params.get("speed") == "fast":
            del params["speed"]
            return create_message_with_fast_fallback(max_retries=max_retries, **params)
        raise
    except (
        anthropic.APIStatusError,
        anthropic.APIConnectionError,
    ) as error:
        if isinstance(error, anthropic.APIStatusError) and error.status_code < 500:
            raise
        if max_attempts > 1:
            return create_message_with_fast_fallback(
                max_retries=max_retries, max_attempts=max_attempts - 1, **params
            )
        raise


message = create_message_with_fast_fallback(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
    speed="fast",
    max_retries=0,
)
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function createMessageWithFastFallback(
  params: Anthropic.MessageCreateParamsNonStreaming,
  requestOptions?: Anthropic.RequestOptions,
  maxAttempts: number = 3
): Promise<Anthropic.Message> {
  try {
    return await client.messages.create(params, requestOptions);
  } catch (e) {
    if (e instanceof Anthropic.RateLimitError && params.speed === "fast") {
      const { speed, ...rest } = params;
      return createMessageWithFastFallback(rest);
    }
    if (
      e instanceof Anthropic.InternalServerError ||
      e instanceof Anthropic.APIConnectionError
    ) {
      if (maxAttempts > 1) {
        return createMessageWithFastFallback(params, undefined, maxAttempts - 1);
      }
    }
    throw e;
  }
}

const message = await createMessageWithFastFallback(
  {
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello" }],
    speed: "fast"
  },
  { maxRetries: 0 }
);
```

## Related

- [Models](./MODELS.md)
- [Prompt Caching](./prompt-caching.md)
- [Rate Limits & Errors](./rate-limits-errors.md)
- [Fast mode (official)](https://platform.claude.com/docs/en/build-with-claude/fast-mode)
