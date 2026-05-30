# Batch API (Message Batches)

> **Last updated:** 2026-05-30

## Overview

The Batch API processes multiple message requests asynchronously. Use it when you have many requests to process and latency isn't critical.

**Endpoint:** `POST /v1/messages/batches`

## Benefits

- Up to 50% cost reduction vs. synchronous requests
- Process up to 10,000 requests per batch
- No rate limit concerns during processing
- Results available for 29 days after creation

## Creating a Batch

```python
import anthropic

client = anthropic.Anthropic()

batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": "request-1",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Translate 'Hello' to Spanish."}],
            },
        },
        {
            "custom_id": "request-2",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Translate 'Goodbye' to French."}],
            },
        },
    ]
)

print(f"Batch ID: {batch.id}")
print(f"Status: {batch.processing_status}")
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const batch = await client.messages.batches.create({
  requests: [
    {
      custom_id: 'request-1',
      params: {
        model: 'claude-sonnet-4-6',
        max_tokens: 1024,
        messages: [{ role: 'user', content: 'Translate "Hello" to Spanish.' }],
      },
    },
  ],
});

console.log(`Batch ID: ${batch.id}`);
```

## Checking Batch Status

```python
batch = client.messages.batches.retrieve(batch_id)
print(f"Status: {batch.processing_status}")
print(f"Requests: {batch.request_counts}")
# request_counts: {processing, succeeded, errored, canceled, expired}
```

## Processing Status Values

| Status | Meaning |
|--------|--------|
| `in_progress` | Batch is being processed |
| `ended` | Processing complete (check individual result statuses) |

## Polling Until Complete

```python
import time

def wait_for_batch(client, batch_id, poll_interval=30):
    while True:
        batch = client.messages.batches.retrieve(batch_id)
        if batch.processing_status == "ended":
            return batch
        print(f"Waiting... {batch.request_counts}")
        time.sleep(poll_interval)

batch = wait_for_batch(client, batch_id)
```

## Retrieving Results

```python
# Stream results
for result in client.messages.batches.results(batch_id):
    print(f"ID: {result.custom_id}, Status: {result.result.type}")
    if result.result.type == "succeeded":
        print(f"Response: {result.result.message.content[0].text}")
    elif result.result.type == "errored":
        print(f"Error: {result.result.error}")
```

```typescript
// TypeScript
const results = await client.messages.batches.results(batch_id);
for await (const result of results) {
  if (result.result.type === 'succeeded') {
    console.log(result.custom_id, result.result.message.content[0].text);
  }
}
```

## Result Types

| Result Type | Meaning |
|------------|--------|
| `succeeded` | Request completed; `result.message` contains response |
| `errored` | Request failed; `result.error` contains error details |
| `canceled` | Batch was canceled before this request processed |
| `expired` | Batch expired (24 hours) before this request processed |

## Limits

| Limit | Value |
|-------|-------|
| Requests per batch | 10,000 max |
| Batch processing time | Up to 24 hours |
| Results retention | 29 days |

## List / Cancel / Delete Batches

```python
# List
batches = client.messages.batches.list()

# Cancel
client.messages.batches.cancel(batch_id)

# Delete
client.messages.batches.delete(batch_id)
```

## Best Practices

- Use `custom_id` values that map back to your data (e.g., database IDs, filenames)
- Process batches asynchronously — don't block your main application waiting for results
- Handle both `succeeded` and `errored` results in your processing logic
- For time-sensitive work, use synchronous API; batches can take hours

## Related

- [Messages API](./messages-api.md)
- [Rate Limits & Errors](./rate-limits-errors.md)
- [Prompt Caching](./prompt-caching.md)
