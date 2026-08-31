# Files API

> **Last updated:** 2026-08-31  
> **Status:** GA (out of beta as of 2026-08-19)

## Overview

The Files API lets you upload files once and reference them by ID across multiple requests. Useful for large documents, PDFs, or images reused in many prompts.

**Base endpoint:** `/v1/files`

## GA Migration Note (2026-08-19)

The Files API is now GA. The `files-api-2025-04-14` beta header is no longer required. The GA response format includes:
- **File expiration**: set `expires_in_seconds` on upload; `expires_at` on the file object
- **Improved pagination**: `page` and `next_page` cursors, plus `ids[]` filter when listing
- Requests that still send the beta header continue to work and return the previous (no-expiration) response format

Use `client.files.*` (not `client.beta.files.*`) in SDK v1.0.0+.

## Upload a File

```python
import anthropic

client = anthropic.Anthropic()

# Upload with optional expiration (GA: no beta header needed)
with open("document.pdf", "rb") as f:
    uploaded = client.files.upload(
        file=("document.pdf", f, "application/pdf"),
        # expires_in_seconds=86400,  # optional: 1-day TTL
    )

file_id = uploaded.id
print(f"Uploaded: {file_id}, expires: {uploaded.expires_at}")
```

```typescript
import Anthropic from '@anthropic-ai/sdk';
import * as fs from 'fs';

const client = new Anthropic();

// GA: no beta header needed
const file = await client.files.upload({
  file: fs.createReadStream('document.pdf'),
  filename: 'document.pdf',
  type: 'application/pdf',
  // expiresInSeconds: 86400,  // optional: 1-day TTL
});

console.log(`File ID: ${file.id}, expires: ${file.expiresAt}`);
```

## Use File in Messages

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "document",
                    "source": {
                        "type": "file",
                        "file_id": file_id,
                    },
                },
                {"type": "text", "text": "Summarize the key findings."},
            ],
        }
    ],
)
```

## List / Retrieve / Delete Files

```python
# List (with pagination and optional ID filter)
files = client.files.list()
for f in files.data:
    print(f"{f.id}: {f.filename} ({f.size} bytes), expires: {f.expires_at}")

# List only specific IDs
specific = client.files.list(ids=["file_abc123", "file_def456"])

# Retrieve metadata
file_info = client.files.retrieve(file_id)

# Delete
client.files.delete(file_id)
```

## Supported File Types

| Type | MIME Type | Use as |
|------|-----------|--------|
| PDF | `application/pdf` | `document` block |
| PNG | `image/png` | `image` block |
| JPEG | `image/jpeg` | `image` block |
| WebP | `image/webp` | `image` block |
| GIF | `image/gif` | `image` block |
| Plain text | `text/plain` | `document` block |

## Benefits over Inline Content

| Feature | Inline Base64 | Files API |
|---------|--------------|----------|
| Request size | Large | Small (just file_id) |
| Reusability | Upload every time | Upload once |
| Network efficiency | Sends full data each request | ID reference only |

## File Size Limits

- Maximum file size: 32 MB

## Batch Processing with Files

```python
# Upload once, query many times
with open("large_doc.pdf", "rb") as f:
    file = client.beta.files.upload(file=("large_doc.pdf", f, "application/pdf"))

requests = [
    {
        "custom_id": f"question-{i}",
        "params": {
            "model": "claude-sonnet-4-6",
            "max_tokens": 512,
            "messages": [{
                "role": "user",
                "content": [
                    {"type": "document", "source": {"type": "file", "file_id": file.id}},
                    {"type": "text", "text": question},
                ],
            }],
        },
    }
    for i, question in enumerate(questions)
]

batch = client.messages.batches.create(requests=requests)
```

## File Expiration

```python
# Upload with 24-hour TTL
with open("temp_doc.pdf", "rb") as f:
    file = client.files.upload(
        file=("temp_doc.pdf", f, "application/pdf"),
        expires_in_seconds=86400,  # 1 day
    )
print(file.expires_at)  # ISO 8601 timestamp
```

Files without `expires_in_seconds` never expire (until manually deleted).

## Gotchas

- **GA as of 2026-08-19**: no beta header required; use `client.files.*` (not `client.beta.files.*`) in SDK v1.0.0+
- **SDK v1.2.0+ (Python) / v0.122.0+ (TypeScript)**: `client.beta.files.*` now uses the GA shape and no longer sends the `files-api-2025-04-14` beta header. Migrate to `client.files.*` to ensure you always get GA response shapes.
- If you send the old `files-api-2025-04-14` beta header, the API returns the previous response format (no `expires_at` field)
- Files are scoped to your API key/organization
- Deleted files immediately become unavailable
- File IDs are stable — safe to store in your database for reuse

## Related

- [PDF Support](./pdf-support.md)
- [Vision](./vision.md)
- [Batch API](./batch-api.md)
- [Prompt Caching](./prompt-caching.md)
