# Rate Limits & Error Handling

> **Last updated:** 2026-05-30

## Rate Limit Headers

Every API response includes rate limit information:

| Header | Description |
|--------|-------------|
| `x-ratelimit-limit-requests` | Max requests per minute |
| `x-ratelimit-limit-tokens` | Max tokens per minute |
| `x-ratelimit-remaining-requests` | Requests remaining in current window |
| `x-ratelimit-remaining-tokens` | Tokens remaining in current window |
| `x-ratelimit-reset-requests` | When request limit resets (RFC 3339) |
| `x-ratelimit-reset-tokens` | When token limit resets (RFC 3339) |
| `retry-after` | Seconds to wait (on 429 responses) |

## Error HTTP Status Codes

| HTTP Status | Error Type | Meaning |
|------------|------------|--------|
| 400 | `invalid_request_error` | Bad request format, invalid parameters |
| 401 | `authentication_error` | Invalid or missing API key |
| 403 | `permission_error` | API key lacks permission |
| 404 | `not_found_error` | Resource not found |
| 429 | `rate_limit_error` | Too many requests |
| 500 | `api_error` | Internal server error |
| 529 | `overloaded_error` | API temporarily overloaded |

## Error Response Format

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\""
  }
}
```

## Python Error Handling

```python
import anthropic

client = anthropic.Anthropic()

try:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello!"}],
    )
except anthropic.AuthenticationError as e:
    print(f"Auth error: {e.status_code} - {e.message}")
except anthropic.RateLimitError as e:
    print(f"Rate limited. Retry after: {e.response.headers.get('retry-after')}s")
except anthropic.APIStatusError as e:
    print(f"API error {e.status_code}: {e.message}")
except anthropic.APIConnectionError as e:
    print(f"Connection error: {e}")
```

## TypeScript Error Handling

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

try {
  const response = await client.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 1024,
    messages: [{ role: 'user', content: 'Hello!' }],
  });
} catch (err) {
  if (err instanceof Anthropic.AuthenticationError) {
    console.error('Auth error:', err.status, err.message);
  } else if (err instanceof Anthropic.RateLimitError) {
    const retryAfter = err.response.headers.get('retry-after');
    console.error(`Rate limited. Retry after ${retryAfter}s`);
  } else if (err instanceof Anthropic.APIError) {
    console.error(`API error ${err.status}:`, err.message);
  } else {
    throw err;
  }
}
```

## Exception Hierarchy (Python)

```
anthropic.APIError
├── anthropic.APIStatusError          (HTTP errors with status code)
│   ├── anthropic.AuthenticationError     (401)
│   ├── anthropic.PermissionDeniedError   (403)
│   ├── anthropic.NotFoundError           (404)
│   ├── anthropic.RateLimitError          (429)
│   └── anthropic.InternalServerError     (500)
└── anthropic.APIConnectionError      (network errors)
    └── anthropic.APITimeoutError     (timeout)
```

## SDK Auto-Retry Configuration

```python
# Python
client = anthropic.Anthropic(
    max_retries=3,       # Default: 2
    timeout=60.0,        # Default: 600s
)
```

```typescript
// TypeScript
const client = new Anthropic({
  maxRetries: 3,     // Default: 2
  timeout: 60000,    // 60 seconds (ms)
});
```

SDK auto-retries on: 429, 500, 529.

## Manual Retry with Backoff

```python
import time

def with_retry(fn, max_retries=3):
    for attempt in range(max_retries):
        try:
            return fn()
        except anthropic.RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait = 2 ** attempt  # 1s, 2s, 4s
            time.sleep(wait)
        except anthropic.InternalServerError:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

## Common Errors and Fixes

### 400 — Roles must alternate
Messages must strictly alternate user/assistant. First must be `user`.

### 401 — Invalid API key
Check `ANTHROPIC_API_KEY` env var or explicit `api_key` parameter.

### 429 — Rate Limited
Check `retry-after` header. SDK handles automatically with default retries.

### 529 — Overloaded
Transient — retry with exponential backoff. SDK handles automatically.

## Related

- [Authentication](./authentication.md)
- [Batch API](./batch-api.md)
- [SDKs](./sdks.md)
