# PDF Support

> **Last updated:** 2026-05-30

## Overview

Claude can read and analyze PDF documents passed as content blocks in messages. PDFs are processed as document inputs with `media_type: "application/pdf"`.

## Inline PDF (Base64)

```python
import anthropic
import base64
from pathlib import Path

client = anthropic.Anthropic()

pdf_data = base64.standard_b64encode(Path("document.pdf").read_bytes()).decode("utf-8")

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
                        "type": "base64",
                        "media_type": "application/pdf",
                        "data": pdf_data,
                    },
                },
                {
                    "type": "text",
                    "text": "Summarize the key points of this document.",
                },
            ],
        }
    ],
)
print(response.content[0].text)
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';
import * as fs from 'fs';

const client = new Anthropic();
const pdfData = fs.readFileSync('document.pdf').toString('base64');

const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 2048,
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'document',
          source: {
            type: 'base64',
            media_type: 'application/pdf',
            data: pdfData,
          },
        },
        { type: 'text', text: 'What are the main conclusions?' },
      ],
    },
  ],
});
```

## PDF via Files API

Upload a PDF using the Files API for reuse across requests:

```python
# Upload once
with open("document.pdf", "rb") as f:
    uploaded = client.beta.files.upload(
        file=("document.pdf", f, "application/pdf"),
    )
file_id = uploaded.id

# Use in messages (no re-upload needed)
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
                {"type": "text", "text": "Analyze this document."},
            ],
        }
    ],
)
```

## Limits

| Limit | Value |
|-------|-------|
| Max PDF size | 32 MB |
| Max pages | 100 pages (some models may differ) |
| Content | Text and images within PDF are both processed |

## Caching PDFs

For large PDFs reused across multiple requests, use prompt caching:

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
                    "source": {"type": "base64", "media_type": "application/pdf", "data": pdf_data},
                    "cache_control": {"type": "ephemeral"},  # Cache the PDF
                },
                {"type": "text", "text": "Answer: What is the revenue for Q3?"},
            ],
        }
    ],
)
```

## Best Practices

- For PDFs used repeatedly, use the Files API to upload once and reference by ID
- Combine with prompt caching for large PDFs to reduce cost on repeated queries
- Keep PDFs under 10MB for reliable processing; larger files may timeout
- For text extraction use cases, consider extracting text first and sending as plain text (cheaper)
- Include specific page references in your prompt if querying a large document ("on page 5...")

## Related

- [Vision](./vision.md)
- [Files API](./files-api.md)
- [Prompt Caching](./prompt-caching.md)
- [Token Counting](./token-counting.md)
