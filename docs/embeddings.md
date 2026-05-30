# Embeddings

> **Last updated:** 2026-05-30

## Overview

Claude itself does not provide an embeddings API. Anthropic's recommended embedding solution is **Voyage AI**, which provides state-of-the-art embedding models optimized for use with Claude.

## Voyage AI (Recommended)

### Installation

```bash
pip install voyageai
```

### Authentication

```bash
export VOYAGE_API_KEY="your-voyage-api-key"
```

### Basic Usage

```python
import voyageai

vo = voyageai.Client()

embedding = vo.embed(
    texts=["The quick brown fox"],
    model="voyage-3",
    input_type="document",  # or "query"
)
print(f"Dimensions: {len(embedding.embeddings[0])}")
```

### Embedding Types

| `input_type` | Use For |
|------------|--------|
| `"query"` | Search queries (questions being asked) |
| `"document"` | Documents being indexed |
| `None` | Symmetric similarity |

Always use `"query"` for search queries and `"document"` for indexed content.

## Available Voyage Models

| Model | Context Length | Dimensions | Best For |
|-------|---------------|-----------|--------|
| `voyage-3` | 32,000 | 1024 | General purpose (recommended) |
| `voyage-3-lite` | 32,000 | 512 | Faster, lower cost |
| `voyage-code-3` | 32,000 | 1024 | Code retrieval |
| `voyage-finance-2` | 32,000 | 1024 | Financial documents |
| `voyage-law-2` | 16,000 | 1024 | Legal documents |
| `voyage-multilingual-2` | 32,000 | 1024 | Non-English text |

## RAG Pipeline with Claude

```python
import voyageai
import anthropic
import numpy as np

vo = voyageai.Client()
client = anthropic.Anthropic()

documents = [
    "Claude is an AI assistant made by Anthropic.",
    "Anthropic was founded in 2021.",
    "Claude models support vision and tool use.",
]

doc_embeddings = vo.embed(texts=documents, model="voyage-3", input_type="document").embeddings

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

def retrieve(query: str, top_k: int = 2) -> list[str]:
    query_embedding = vo.embed(texts=[query], model="voyage-3", input_type="query").embeddings[0]
    scores = [cosine_similarity(query_embedding, doc_emb) for doc_emb in doc_embeddings]
    top_indices = np.argsort(scores)[-top_k:][::-1]
    return [documents[i] for i in top_indices]

def rag_query(question: str) -> str:
    context = "\n\n".join(retrieve(question))
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=f"Answer questions based on this context:\n\n{context}",
        messages=[{"role": "user", "content": question}],
    )
    return response.content[0].text
```

## Via HTTP API

```bash
curl -X POST https://api.voyageai.com/v1/embeddings \
  -H "Authorization: Bearer $VOYAGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input": ["Document to embed"], "model": "voyage-3", "input_type": "document"}'
```

## Voyage API Limits

| Limit | Value |
|-------|-------|
| Max texts per request | 128 |
| Embedding normalized | Yes (unit vectors) |

Normalized embeddings: dot product = cosine similarity.

## Related

- [Agent Patterns](./agent-patterns.md)
- [Prompt Caching](./prompt-caching.md)
- [Models](./MODELS.md)
