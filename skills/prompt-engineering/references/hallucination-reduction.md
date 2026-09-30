# Hallucination Reduction for Claude

## Core Principle

Hallucinations happen when Claude generates plausible-sounding but incorrect information. The most effective mitigation combines multiple techniques.

## Technique 1: Permission to Say "I Don't Know"

Explicitly grant Claude permission to admit uncertainty. Without this, Claude may fabricate answers to be "helpful."

```
If you don't know the answer or aren't confident, say "I'm not sure" and explain what you'd need to give a confident answer. Never guess at facts, statistics, or citations.
```

## Technique 2: Direct Quote Grounding

For tasks involving documents (especially >20K tokens), require Claude to extract verbatim quotes before analyzing.

```
<instructions>
1. First, extract the 3-5 most relevant direct quotes from the document that relate to the question.
2. Wrap each quote in <quote> tags with the section/page reference.
3. Then answer the question using ONLY information from those quotes.
4. If no relevant quotes exist, say so.
</instructions>
```

This grounds responses in actual source material rather than Claude's parametric knowledge.

## Technique 3: Citation Verification

Make Claude's claims auditable by requiring citations, then self-verifying them.

```
For each factual claim in your response:
1. Cite the specific source (document section, page, URL)
2. After drafting your full response, review each citation
3. If you cannot find a supporting quote for a claim, retract it and note "[UNVERIFIED - removed]"
```

## Technique 4: External Knowledge Restriction

When Claude should ONLY use provided documents:

```
Answer using ONLY the information in the provided documents. Do not use your general knowledge or training data. If the documents don't contain enough information to answer, say "The provided documents do not contain sufficient information to answer this question."
```

## Technique 5: Chain-of-Thought Verification

Have Claude show its reasoning to expose faulty logic:

```
Before giving your final answer:
1. State the key facts you're relying on
2. Explain your reasoning step by step
3. Identify any assumptions you're making
4. Check: could any of these assumptions be wrong?
Then provide your final answer.
```

## Technique 6: Best-of-N Verification

For critical outputs, run the same prompt multiple times and compare:

```python
# API pattern: run 3 times and compare
responses = []
for _ in range(3):
    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        temperature=0.3,  # Low but not zero for slight variation
        messages=[{"role": "user", "content": prompt}]
    )
    responses.append(response)

# Compare outputs — inconsistencies flag potential hallucinations
```

## Technique 7: Temperature Tuning

- **temperature=0**: Most deterministic, best for factual tasks
- **temperature=0.3-0.5**: Slight variation, good for Best-of-N
- **temperature=0.7-1.0**: More creative, higher hallucination risk

Rule of thumb: use lower temperature for factual/analytical tasks, higher for creative tasks.

## Technique 8: Iterative Refinement

Use Claude's output as input for a verification pass:

```
<draft>
[Claude's initial response]
</draft>

Review the draft above. For each factual claim:
- Is it supported by the source material?
- Is it accurate based on your knowledge?
- Flag any claim you're less than 90% confident about with [NEEDS VERIFICATION]
Produce a corrected version.
```

## Combining Techniques (Recommended for Production)

For high-stakes applications, layer multiple techniques:

```
<instructions>
You are answering questions based on the provided documents.

Rules:
1. Use ONLY information from the provided documents (no external knowledge)
2. For each claim, cite the source section in parentheses
3. If unsure, say "I'm not sure" rather than guessing
4. After completing your response, verify each citation exists in the source
5. Retract any claim you cannot verify

If the documents don't contain the answer, say so explicitly.
</instructions>
```
