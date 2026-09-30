# Production Optimization for Claude Prompts

## Model Selection

Choose the right model for the task — this often has more impact than prompt changes.

| Model | Best for | Trade-off |
|---|---|---|
| Claude Opus 4.6 | Most complex reasoning, coding, multi-step agents | Highest cost, highest latency |
| Claude Sonnet 4.5 | Balanced: strong reasoning, good speed, production workloads | Mid cost, mid latency |
| Claude Haiku 4.5 | High-volume, simple tasks, near-instant responses | Lowest cost, lowest latency |

**Decision rule:** Start with the cheapest model that passes your evals. Only upgrade if quality is insufficient.

## Latency Reduction

Ordered by impact, try these in sequence:

### 1. Choose a Faster Model
Haiku is significantly faster than Sonnet, which is faster than Opus. If quality is acceptable, downgrade.

### 2. Reduce Token Count
Fewer input + output tokens = faster response.

- **Input:** Remove filler, redundant instructions, and context Claude doesn't need. Test a minimal prompt.
- **Output:** Set `max_tokens` to the minimum needed. Add explicit length constraints: "Respond in 2 sentences" or "Return only the JSON object."
- Use prefilling to skip preamble (e.g., start assistant response with `{` for JSON).

### 3. Use Streaming
Stream responses so users see output in real-time rather than waiting for the full response:

```python
with client.messages.stream(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{"role": "user", "content": prompt}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

This reduces *perceived* latency significantly even if total generation time is the same.

### 4. Parallelize Independent Tasks
If you need multiple outputs (e.g., summary + sentiment + entities), run them as parallel API calls rather than one sequential prompt.

### 5. Avoid Unnecessary Chain of Thought
CoT adds tokens and latency. Only use it when reasoning quality actually improves.

## Prompt Caching

Cache long, reusable prompt prefixes to reduce cost and latency on repeated calls.

```python
response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are a legal document analyst...",  # Long system prompt
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": user_question}]
)
```

**When to cache:**
- System prompts >1024 tokens that stay the same across requests
- Large reference documents (e.g., a 50-page contract analyzed by many users)
- Few-shot example sets that don't change

**Pricing benefit:** Cached tokens cost ~90% less than uncached tokens. TTFT also improves.

**Cache lifetime:** Ephemeral caches last ~5 minutes and refresh on each use.

## Cost Optimization

### Token-Efficient Prompting
- Remove "be helpful" / "be thorough" — Claude does this by default
- Use XML tags instead of verbose delimiters
- Use short variable names in templates: `{q}` vs `{user_question_text}`
- Limit output length explicitly

### Batch API
For non-time-sensitive workloads (analysis, classification, bulk processing), use the Batch API for 50% cost reduction:

```python
# Create a batch of requests
batch = client.batches.create(
    requests=[
        {"custom_id": "req-1", "params": {"model": "...", "messages": [...]}},
        {"custom_id": "req-2", "params": {"model": "...", "messages": [...]}},
    ]
)
```

Batches complete within 24 hours. Ideal for: daily report generation, bulk classification, dataset labeling.

### Structured Outputs
Use the `response_format` parameter for guaranteed JSON schema compliance instead of prompt-engineering JSON output:

```python
response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Analyze this feedback..."}],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "feedback_analysis",
            "schema": {
                "type": "object",
                "properties": {
                    "sentiment": {"type": "string", "enum": ["positive", "negative", "neutral"]},
                    "topics": {"type": "array", "items": {"type": "string"}},
                    "urgency": {"type": "integer", "minimum": 1, "maximum": 5}
                },
                "required": ["sentiment", "topics", "urgency"]
            }
        }
    }
)
```

This eliminates format-related retries and parsing failures.

## Extended Thinking Budget

For models with extended thinking, manage the thinking budget:

- **Minimum:** 1024 tokens — start here
- Increase incrementally based on task complexity
- Monitor: are more thinking tokens actually improving output quality?
- For production, set the minimum budget that passes evals

```python
response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=8000,
    thinking={
        "type": "enabled",
        "budget_tokens": 4000  # Start low, increase if needed
    },
    messages=[{"role": "user", "content": prompt}]
)
```
