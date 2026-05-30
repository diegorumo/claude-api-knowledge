# Agent Patterns & Best Practices

> **Last updated:** 2026-05-30  
> **Source:** Anthropic Cookbook — "Building Effective Agents" by Erik Schluntz and Barry Zhang

## Core Principle

Build agents as **minimal, composable workflows** rather than complex frameworks. The most reliable agents are often simple loops with clear tool definitions.

## Pattern 1: Prompt Chaining

Decompose complex tasks into sequential steps where each step's output feeds the next.

```python
import anthropic

client = anthropic.Anthropic()

def llm_call(prompt: str, model: str = "claude-sonnet-4-6") -> str:
    response = client.messages.create(
        model=model,
        max_tokens=2048,
        messages=[{"role": "user", "content": prompt}],
    )
    return response.content[0].text

def chain(input_text: str, prompts: list[str]) -> str:
    result = input_text
    for i, prompt in enumerate(prompts, 1):
        print(f"Step {i}...")
        result = llm_call(f"{prompt}\n\nInput: {result}")
    return result

output = chain(
    raw_data,
    [
        "Extract all names and dates from this text.",
        "Normalize the date formats to ISO 8601.",
        "Sort the entries alphabetically by name.",
        "Format as a JSON array.",
    ]
)
```

**When to use:** Multi-step transformations, document processing pipelines.

## Pattern 2: Parallelization

Run independent subtasks concurrently to reduce total latency.

```python
from concurrent.futures import ThreadPoolExecutor

def parallel(prompt: str, inputs: list[str], n_workers: int = 3) -> list[str]:
    with ThreadPoolExecutor(max_workers=n_workers) as executor:
        futures = [
            executor.submit(llm_call, f"{prompt}\n\nInput: {x}")
            for x in inputs
        ]
        return [f.result() for f in futures]

# Analyze multiple documents simultaneously
summaries = parallel(
    "Summarize this document in 3 bullet points:",
    documents,
    n_workers=5,
)
```

**When to use:** Independent subtasks (sentiment analysis across many texts, parallel research).

## Pattern 3: Dynamic Routing

Classify inputs and route to specialized handlers.

```python
import re

def extract_xml(text: str, tag: str) -> str:
    match = re.search(f"<{tag}>(.*?)</{tag}>", text, re.DOTALL)
    return match.group(1).strip() if match else ""

def route(input_text: str, routes: dict[str, str]) -> str:
    selector_prompt = f"""
Analyze this input and select the most appropriate handler from: {list(routes.keys())}

Return:
<reasoning>Brief explanation</reasoning>
<selection>handler_name</selection>

Input: {input_text}
""".strip()

    response = llm_call(selector_prompt)
    route_key = extract_xml(response, "selection").strip().lower()
    selected_prompt = routes.get(route_key, routes.get("default", "Handle this: "))
    return llm_call(f"{selected_prompt}\n\nInput: {input_text}")

routes = {
    "billing": "You are a billing specialist. Resolve this billing issue:",
    "technical": "You are a technical support engineer. Diagnose this problem:",
    "account": "You are an account manager. Help with this account question:",
}

response = route(user_query, routes)
```

**When to use:** Customer support, content moderation, domain-specialized responses.

## Pattern 4: Evaluator-Optimizer Loop

Generate output, evaluate it, refine — repeat until quality threshold met.

```python
def generate_and_refine(task: str, quality_criteria: str, max_iterations: int = 3) -> str:
    response = llm_call(f"Complete this task: {task}")
    
    for i in range(max_iterations):
        evaluation = llm_call(f"""
Evaluate this response against: {quality_criteria}

Response: {response}

Return: <score>1-10</score><feedback>specific improvements</feedback>
""")
        score = int(extract_xml(evaluation, "score"))
        if score >= 8:
            break
        feedback = extract_xml(evaluation, "feedback")
        response = llm_call(f"Improve this response based on feedback.\n\nTask: {task}\nCurrent: {response}\nFeedback: {feedback}\n\nImproved version:")
    
    return response
```

**When to use:** Content quality improvement, code review + fix loops, translation QA.

## Basic Agentic Tool Loop

```python
def run_agent(
    client: anthropic.Anthropic,
    tools: list,
    tool_functions: dict,
    user_message: str,
    model: str = "claude-sonnet-4-6",
    max_iterations: int = 10,
) -> str:
    messages = [{"role": "user", "content": user_message}]
    
    for iteration in range(max_iterations):
        response = client.messages.create(
            model=model,
            max_tokens=4096,
            tools=tools,
            messages=messages,
        )
        
        if response.stop_reason == "end_turn":
            return next((b.text for b in response.content if b.type == "text"), "Done.")
        
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    fn = tool_functions.get(block.name)
                    try:
                        result = fn(**block.input) if fn else "Tool not found"
                        tool_results.append({"type": "tool_result", "tool_use_id": block.id, "content": str(result)})
                    except Exception as e:
                        tool_results.append({"type": "tool_result", "tool_use_id": block.id, "content": f"Error: {e}", "is_error": True})
            
            messages.append({"role": "user", "content": tool_results})
    
    raise RuntimeError(f"Agent exceeded max iterations ({max_iterations})")
```

## When to Use Which Pattern

| Scenario | Recommended Pattern |
|----------|--------------------|
| Fixed, known steps | Prompt chaining |
| Independent subtasks | Parallelization |
| Input categorization | Routing |
| Open-ended research | Agentic loop with tools |
| Creative + quality bar | Evaluator-optimizer |
| Very complex tasks | Orchestrator-workers |

## Related

- [Tool Use](./tool-use.md)
- [Managed Agents](./managed-agents.md)
- [Streaming](./streaming.md)
- [Prompt Engineering](./prompt-engineering.md)
