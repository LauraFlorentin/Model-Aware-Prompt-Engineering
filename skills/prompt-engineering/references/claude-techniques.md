# Claude-Specific Prompt Engineering Techniques

## XML Tags

Claude responds exceptionally well to XML tags for structuring prompts. Use them to separate sections, wrap examples, and define input/output boundaries.

```
<context>
You are reviewing a legal contract for a small business owner.
</context>

<instructions>
Identify the 3 most important clauses and explain each in plain English.
</instructions>

<formatting>
Use ## headings for each clause. Keep explanations under 50 words each.
</formatting>
```

**Best tags to use:** `<instructions>`, `<context>`, `<examples>`, `<input>`, `<output>`, `<constraints>`, `<formatting>`, `<persona>`, `<task>`. Invent domain-specific tags as needed (e.g. `<code_to_review>`, `<customer_email>`).

## Chain of Thought (CoT)

Ask Claude to reason step by step for complex tasks. For Claude 4.x, use "consider" or "evaluate" instead of "think" when extended thinking is disabled (Opus 4.5 is sensitive to the word "think").

```
Before answering, consider step by step:
1. What are the key facts?
2. What assumptions am I making?
3. What's the strongest counterargument?
Then provide your final answer.
```

**When to use CoT:** Math, logic, multi-step analysis, ambiguous questions, code debugging.
**When to skip:** Simple factual lookups, creative writing, translation.

## Extended Thinking

For Claude 4.x models with extended thinking enabled, guide the thinking process:

```
Before responding, use your thinking to:
- Evaluate the quality of the search results
- Identify gaps or conflicting information
- Plan the optimal response structure
Then provide your answer.
```

## Prefilling Claude's Response

In the API, start Claude's response to steer format or tone:

```json
{
  "messages": [
    {"role": "user", "content": "Analyze this data..."},
    {"role": "assistant", "content": "## Analysis\n\n"}
  ]
}
```

Use prefilling to: enforce JSON output (`{`), set a specific format, continue a pattern, or prevent preamble.

## Multishot Prompting (Few-Shot Examples)

Provide 2-5 input/output examples. Claude 4.x models pay very close attention to example details, so ensure they precisely match desired behavior.

```
<examples>
<example>
<input>The product arrived broken and customer service was unhelpful.</input>
<output>{"sentiment": "negative", "topics": ["product_quality", "customer_service"], "urgency": "high"}</output>
</example>
<example>
<input>Love the new update! The dark mode is gorgeous.</input>
<output>{"sentiment": "positive", "topics": ["product_update", "ui_design"], "urgency": "low"}</output>
</example>
</examples>
```

**Tip:** Include edge cases and negative examples (what NOT to do) — they define boundaries more effectively than positive examples alone.

## Role / System Prompts

Assign Claude a role with specific expertise, constraints, and success criteria:

```
You are a senior Python code reviewer at a fintech company.

Your priorities:
1. Security vulnerabilities (SQL injection, XSS, auth bypasses)
2. Performance issues (N+1 queries, memory leaks)
3. Code readability and maintainability

You NEVER suggest stylistic changes unless they affect readability.
You ALWAYS cite the specific line numbers.
```

**Claude 4.x note:** These models follow instructions precisely. Explicitly request "above and beyond" behavior if desired, and provide motivation/context for why instructions matter.

## Prompt Chaining

Break complex tasks into sequential prompts where each step's output feeds the next:

```
Step 1: Extract key claims from the article → list of claims
Step 2: For each claim, search for supporting/contradicting evidence → evidence map
Step 3: Rate each claim's reliability based on evidence → scored claims
Step 4: Write final fact-check report using scored claims → report
```

Use chaining when: tasks have 3+ distinct phases, intermediate validation helps, or total context would exceed limits.

## Long Context Tips

When working with large documents:
- Place the document BEFORE the instructions (Claude processes context better this way)
- Use explicit references: "In the document above, find..."
- For very long inputs, ask Claude to first extract relevant sections, then analyze

## Output Constraints

Be precise about format, length, and structure:

```
Respond in exactly this JSON schema:
{
  "summary": "string, max 100 words",
  "key_points": ["string array, 3-5 items"],
  "confidence": "number 0-1",
  "sources_needed": "boolean"
}

Do not include any text outside the JSON object.
```

## Uncertainty Handling

Always include an uncertainty clause for production prompts:

```
If you are unsure about any fact, say so explicitly rather than guessing.
If the input is ambiguous, state your interpretation before proceeding.
If you cannot complete the task with the given information, explain what's missing.
```
