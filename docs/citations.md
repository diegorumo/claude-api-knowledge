# Citations

> **Last updated:** 2026-05-30  
> **Status:** Beta (`anthropic-beta: citations-2024-11-06`)

## Overview

The Citations feature enables Claude to provide inline citations referencing source documents passed in the request.

## Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "document",
                    "source": {
                        "type": "text",
                        "media_type": "text/plain",
                        "data": """Anthropic was founded in 2021 by Dario Amodei,
                        Daniela Amodei, and other former OpenAI researchers.""",
                    },
                    "title": "Company History",
                    "citations": {"enabled": True},
                },
                {
                    "type": "text",
                    "text": "When was Anthropic founded and by whom?",
                },
            ],
        }
    ],
    betas=["citations-2024-11-06"],
)

for block in response.content:
    if block.type == "text":
        print(block.text)
        if hasattr(block, "citations") and block.citations:
            for citation in block.citations:
                print(f"  [Source: {citation.document_title}]")
```

## TypeScript Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

const response = await client.beta.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 2048,
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'document',
          source: {
            type: 'text',
            media_type: 'text/plain',
            data: 'Claude is an AI assistant. It was first released in 2023.',
          },
          title: 'Claude Overview',
          citations: { enabled: true },
        },
        { type: 'text', text: 'When was Claude first released?' },
      ],
    },
  ],
  betas: ['citations-2024-11-06'],
});
```

## Citation Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `"document_citation"` |
| `document_index` | integer | Index of the source document |
| `document_title` | string | Title of the source document |
| `start_char_index` | integer | Start position in source |
| `end_char_index` | integer | End position in source |
| `cited_text` | string | The actual text being cited |

## Supported Document Types

- Plain text (`text/plain`)
- PDF (`application/pdf`)
- HTML (`text/html`) — beta
- Markdown (`text/markdown`) — beta

## PDF Citations

```python
import base64
pdf_data = base64.standard_b64encode(open("report.pdf", "rb").read()).decode()

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "document",
                "source": {"type": "base64", "media_type": "application/pdf", "data": pdf_data},
                "title": "Annual Report 2025",
                "citations": {"enabled": True},
            },
            {"type": "text", "text": "What were the key risks mentioned?"},
        ],
    }],
    betas=["citations-2024-11-06"],
)
```

## Gotchas

- Requires `citations-2024-11-06` beta header
- Not all text blocks will have citations — only claims traceable to source documents
- Document `title` field is optional but recommended for readable citation output

## Use Cases

- Legal document analysis with source attribution
- Research synthesis across multiple papers
- Customer support with policy document citations
- Financial analysis referencing specific reports
- Any RAG application requiring transparent sourcing

## Related

- [PDF Support](./pdf-support.md)
- [Files API](./files-api.md)
- [Prompt Engineering](./prompt-engineering.md)
