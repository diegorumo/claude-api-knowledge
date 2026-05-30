# Files API

> **Last updated:** 2026-05-30  
> **Status:** Beta

## Overview

The Files API lets you upload files once and reference them by ID across multiple requests. Useful for large documents, PDFs, or images reused in many prompts.

**Base endpoint:** `/v1/files`

## Upload a File

```python
import anthropic

client = anthropic.Anthropic()

with open("document.pdf", "rb") as f:
    uploaded = client.beta.files.upload(
        file=("document.pdf", f, "application/pdf"),
    )

file_id = uploaded.id
print(f"Uploaded: {file_id}")
```

```typescript
import Anthropic from '@anthropic-ai/sdk';
import * as fs from 'fs';

const client = new Anthropic();

const file = await client.beta.files.upload({
  file: fs.createReadStream('document.pdf'),
  filename: 'document.pdf',
  type: 'application/pdf',
});

console.log(`File ID: ${file.id}`);
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
# List
files = client.beta.files.list()
for f in files.data:
    print(f"{f.id}: {f.filename} ({f.size} bytes)")

# Retrieve metadata
file_info = client.beta.files.retrieve(file_id)

# Delete
client.beta.files.delete(file_id)
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

## Gotchas

- Files API is in beta — enable via `anthropic-beta` header or SDK beta client
- Files are scoped to your API key/organization
- Deleted files immediately become unavailable
- File IDs are stable — safe to store in your database for reuse

## Related

- [PDF Support](./pdf-support.md)
- [Vision](./vision.md)
- [Batch API](./batch-api.md)
- [Prompt Caching](./prompt-caching.md)
