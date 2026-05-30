# Prompt Engineering Guide

> **Last updated:** 2026-05-30

## Core Principles

### 1. Be Specific and Direct

```
# Weak
"Tell me about databases."

# Better
"Explain the difference between relational and document databases in 3 bullet points,
focusing on when to choose each."
```

### 2. Use System Prompts for Persistent Context

```python
client.messages.create(
    model="claude-sonnet-4-6",
    system="""You are a senior Python engineer specializing in FastAPI.

Your guidelines:
- Prefer async patterns
- Always include type hints
- Write tests alongside implementations
- Follow PEP 8""",
    messages=[{"role": "user", "content": "How do I add authentication?"}],
)
```

### 3. Structured Output via XML Tags

```python
prompt = """Analyze this code and provide:
<issues>List any bugs or problems</issues>
<improvements>List potential improvements</improvements>
<rating>Score 1-10</rating>

Code: {code}"""
```

### 4. Chain of Thought

```python
prompt = """Think through this problem step by step before answering.

Problem: {problem}

First, reason through your approach:
<thinking>...</thinking>

Then provide your final answer:
<answer>...</answer>"""
```

### 5. Few-Shot Examples

```python
prompt = """Classify the sentiment of customer reviews.

Examples:
Review: "The product works great!" → positive
Review: "Broken on arrival, terrible." → negative
Review: "It arrived on time." → neutral

Now classify:
Review: "{review}" →"""
```

## JSON Output

### Method 1: Response Prefilling

```python
messages = [
    {"role": "user", "content": f"Extract entities from: {text}. Return JSON only."},
    {"role": "assistant", "content": "{"},  # Prefill forces JSON start
]
response = client.messages.create(model="claude-sonnet-4-6", max_tokens=1024, messages=messages)
full_json = "{" + response.content[0].text
```

### Method 2: Explicit JSON Instruction

```python
prompt = """Extract the following and return as valid JSON:
- person_name (string)
- date (ISO 8601)
- location (string)

Text: {text}

Return only the JSON object, no other text."""
```

### Method 3: Structured Outputs SDK Helper

```python
import pydantic

class Extraction(pydantic.BaseModel):
    person_name: str
    date: str
    location: str

result = client.messages.parse(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": f"Extract info from: {text}"}],
    output_format=Extraction,
)
print(result.parsed_output.person_name)
```

## System Prompt Patterns

### Persona with Constraints

```
You are Alex, a friendly customer support agent for Acme Corp.

You MUST:
- Always greet users warmly
- Escalate billing issues to billing@acme.com

You MUST NOT:
- Promise features not in the product roadmap
- Diagnose specific technical issues without reproduction steps
```

### Format Instructions

```
You are a code review assistant.

Format all responses as:
## Summary
[1-2 sentence overview]

## Issues Found
[Numbered list, or "No issues found"]

## Suggestions
[Bulleted improvements]

## Score: X/10
```

## Context Window Management

For long conversations, summarize older context:

```python
SUMMARY_THRESHOLD = 150_000

def maybe_summarize(messages, client, model):
    result = client.messages.count_tokens(model=model, messages=messages)
    if result.input_tokens < SUMMARY_THRESHOLD:
        return messages
    
    to_summarize = messages[:-4]
    summary_response = client.messages.create(
        model=model,
        max_tokens=2048,
        messages=[
            *to_summarize,
            {"role": "user", "content": "Summarize our conversation so far in 3-5 sentences."},
        ],
    )
    summary = summary_response.content[0].text
    
    return [
        {"role": "user", "content": f"[Conversation summary: {summary}]"},
        {"role": "assistant", "content": "Understood. Continuing from where we left off."},
        *messages[-4:],
    ]
```

## Temperature Guide

| Temperature | Use For |
|------------|--------|
| 0.0 | Factual Q&A, code generation, data extraction |
| 0.3–0.5 | Analysis, summaries, classification |
| 0.7–0.9 | Creative writing, brainstorming, ideation |
| 1.0 | Maximum creativity (may be less coherent) |

## Common Gotchas

- **Ambiguous instructions:** "Be brief" → "Respond in 2-3 sentences"
- **Conflicting instructions:** Keep system prompt and user instructions consistent
- **Role alternation:** Messages must strictly alternate user/assistant
- **Over-instructing:** Shorter, clearer prompts often outperform lengthy instruction sets

## Related

- [Messages API](./messages-api.md)
- [Extended Thinking](./extended-thinking.md)
- [Tool Use](./tool-use.md)
- [Agent Patterns](./agent-patterns.md)
