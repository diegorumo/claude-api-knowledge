# Vision (Image Inputs)

> **Last updated:** 2026-05-30

## Overview

All Claude models support image inputs via the Messages API. Images can be sent as base64-encoded data or via URL.

## Supported Formats

| Format | MIME Type |
|--------|----------|
| JPEG | `image/jpeg` |
| PNG | `image/png` |
| GIF | `image/gif` |
| WebP | `image/webp` |

## Size Limits

- **Max image size:** 5 MB per image (base64 or URL)
- **Max images per request:** 20 (model-dependent)
- Images larger than ~1568px on the longest side are automatically resized internally

## Base64 Image Input

```python
import anthropic
import base64
from pathlib import Path

client = anthropic.Anthropic()

# Load and encode image
image_data = base64.standard_b64encode(Path("image.png").read_bytes()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "What is in this image?",
                },
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/png",
                        "data": image_data,
                    },
                },
            ],
        }
    ],
)
print(response.content[0].text)
```

## URL Image Input

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image."},
                {
                    "type": "image",
                    "source": {
                        "type": "url",
                        "url": "https://example.com/photo.jpg",
                    },
                },
            ],
        }
    ],
)
```

## TypeScript Examples

```typescript
import Anthropic from '@anthropic-ai/sdk';
import * as fs from 'fs';

const client = new Anthropic();

// Base64 image
const imageData = fs.readFileSync('image.png').toString('base64');

const response = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [
    {
      role: 'user',
      content: [
        { type: 'text', text: 'What is in this image?' },
        {
          type: 'image',
          source: {
            type: 'base64',
            media_type: 'image/png',
            data: imageData,
          },
        },
      ],
    },
  ],
});

// URL image
const urlResponse = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  messages: [
    {
      role: 'user',
      content: [
        { type: 'text', text: 'Describe this.' },
        {
          type: 'image',
          source: { type: 'url', url: 'https://example.com/photo.jpg' },
        },
      ],
    },
  ],
});
```

## Multiple Images

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Compare these two images:"},
                {
                    "type": "image",
                    "source": {"type": "url", "url": "https://example.com/image1.jpg"},
                },
                {"type": "text", "text": "and"},
                {
                    "type": "image",
                    "source": {"type": "url", "url": "https://example.com/image2.jpg"},
                },
                {"type": "text", "text": "What are the key differences?"},
            ],
        }
    ],
)
```

## Best Practices

- Use URL sources when images are publicly accessible — avoids base64 encoding overhead in request body
- For repeated analysis of the same image, combine with prompt caching to cache the image tokens
- Place images close to the relevant text for better context association
- For document/chart analysis, higher resolution improves accuracy
- Include descriptive text alongside images to guide Claude's analysis

## Gotchas

- URL images must be publicly accessible (no auth required)
- HTTPS URLs are preferred; HTTP may not work in all environments
- GIF support is for static GIFs only — animated GIFs are processed as a single frame
- Very large images (>5MB) will be rejected with a 400 error

## Related

- [Messages API](./messages-api.md)
- [PDF Support](./pdf-support.md)
- [Token Counting](./token-counting.md)
- [Prompt Caching](./prompt-caching.md)
