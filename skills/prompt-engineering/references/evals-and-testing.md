# Evaluations & Testing for Claude Prompts

## Why Evals Matter

Prompt engineering without evals is guessing. Evals turn "this feels better" into "this scores 12% higher on accuracy." Always establish evals before optimizing.

## Step 1: Define Success Criteria

Before writing any prompt, answer these questions:

| Question | Example |
|---|---|
| What is the task? | Classify customer emails into categories |
| What does a perfect output look like? | Correct category, correct confidence score, correct escalation flag |
| What are the failure modes? | Wrong category, hallucinated categories, missing escalation |
| How will you measure success? | Accuracy %, false positive rate, latency |

### Types of Success Criteria

- **Exact match:** Output must exactly match expected (e.g., classification label, yes/no)
- **Semantic match:** Output must convey the same meaning (e.g., summaries, explanations)
- **Format compliance:** Output must match a schema (e.g., valid JSON, required fields present)
- **Constraint satisfaction:** Output must stay within bounds (e.g., word count, tone, no hallucinations)
- **Composite:** Weighted combination of the above

## Step 2: Build a Test Set

Create a representative set of inputs with expected outputs.

### Test Set Structure

```json
[
  {
    "id": "test-001",
    "input": "I've been charged twice for my subscription this month.",
    "expected_output": {
      "category": "billing",
      "urgency": "high",
      "escalate": true
    },
    "tags": ["billing", "duplicate-charge", "edge-case"]
  },
  {
    "id": "test-002",
    "input": "How do I change my password?",
    "expected_output": {
      "category": "account",
      "urgency": "low",
      "escalate": false
    },
    "tags": ["account", "common"]
  }
]
```

### Test Set Best Practices

- **Minimum 20 test cases** for meaningful signal, 50-100+ for production prompts
- **Include edge cases:** ambiguous inputs, empty inputs, adversarial inputs, very long inputs
- **Include negative examples:** inputs that should be rejected or flagged
- **Tag test cases** so you can analyze performance by category
- **Version your test sets** alongside your prompts

## Step 3: Build a Scorer

Match your scorer to your success criteria:

### Exact Match Scorer
```python
def score_exact(expected, actual):
    return 1.0 if expected == actual else 0.0
```

### Partial Match Scorer (for multi-field outputs)
```python
def score_partial(expected, actual):
    fields = ["category", "urgency", "escalate"]
    correct = sum(1 for f in fields if expected.get(f) == actual.get(f))
    return correct / len(fields)
```

### LLM-as-Judge Scorer (for open-ended outputs)
Use Claude itself to evaluate quality:

```python
judge_prompt = """
Rate the following response on a scale of 1-5 for each criterion.
Return ONLY a JSON object.

<criteria>
- Accuracy: Are all facts correct? (1=major errors, 5=fully accurate)
- Completeness: Are all key points addressed? (1=missing most, 5=comprehensive)
- Clarity: Is the response clear and well-structured? (1=confusing, 5=crystal clear)
</criteria>

<question>{question}</question>
<response>{response}</response>
<reference_answer>{expected}</reference_answer>
"""
```

**Tip:** LLM-as-judge works well for subjective quality but can miss factual errors. Combine with exact-match scoring where possible.

## Step 4: Run Evals

### Basic Eval Loop

```python
import json

results = []
for test_case in test_set:
    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=1024,
        messages=[{"role": "user", "content": test_case["input"]}],
        system=system_prompt
    )
    actual = parse_response(response)
    score = score_partial(test_case["expected_output"], actual)
    results.append({
        "id": test_case["id"],
        "score": score,
        "tags": test_case["tags"],
        "actual": actual
    })

# Aggregate
avg_score = sum(r["score"] for r in results) / len(results)
print(f"Average score: {avg_score:.2%}")

# Breakdown by tag
from collections import defaultdict
by_tag = defaultdict(list)
for r in results:
    for tag in r["tags"]:
        by_tag[tag].append(r["score"])

for tag, scores in by_tag.items():
    print(f"  {tag}: {sum(scores)/len(scores):.2%}")
```

### Anthropic Console Evaluation Tool

The Anthropic Console includes a built-in evaluation tool. Upload test cases, run prompts, and compare versions side-by-side. Use it for:
- A/B testing prompt variants
- Tracking performance over time
- Sharing results with your team

## Step 5: Iterate

The eval-driven improvement loop:

1. **Baseline:** Run evals on your current prompt → record scores
2. **Diagnose:** Look at the *lowest-scoring* test cases. What pattern do failures share?
3. **Hypothesize:** What prompt change would fix the failure pattern?
4. **Change:** Make ONE change at a time
5. **Re-eval:** Run the same test set → compare to baseline
6. **Keep or revert:** Only keep changes that improve scores without regressions

**Common diagnostic patterns:**
- Failures clustered on one tag → add examples or instructions for that category
- Random failures across tags → prompt is ambiguous, add more structure
- Consistent off-by-one errors → output format is unclear, add a template
- High variance across runs → lower temperature, add more constraints
