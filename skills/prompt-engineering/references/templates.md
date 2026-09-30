# Prompt Templates

Reusable templates for common prompt engineering tasks. Copy and customize for specific use cases.

## Template 1: Task Prompt (General Purpose)

```
<context>
[Background information Claude needs to understand the task]
</context>

<instructions>
[Clear, specific description of what to do]
</instructions>

<constraints>
- [Length/format requirements]
- [What to include/exclude]
- [Tone and audience]
</constraints>

<input>
[The actual content to process]
</input>
```

## Template 2: Analysis Prompt

```
Analyze the following [content type] and provide:

1. **Summary**: [length] overview of the key points
2. **Strengths**: Top [N] strengths with specific evidence
3. **Weaknesses**: Top [N] weaknesses with specific evidence
4. **Recommendations**: [N] actionable next steps, prioritized by impact

Audience: [who will read this]
Tone: [formal/conversational/technical]

<input>
[content to analyze]
</input>
```

## Template 3: Extraction Prompt

```
Extract the following fields from the provided [document type].
Return ONLY valid JSON matching this schema — no additional text.

{
  "field1": "type — description",
  "field2": "type — description",
  "field3": ["type — description"]
}

Rules:
- If a field is not found, use null
- If a field is ambiguous, use your best interpretation and set "confidence": "low"
- Dates in ISO 8601 format
- Numbers as integers/floats, not strings

<document>
[content]
</document>
```

## Template 4: Rewrite / Transform Prompt

```
Rewrite the following [content type] to meet these requirements:

Target audience: [who]
Tone: [e.g., professional, casual, academic]
Length: [word count or relative — "half the original length"]
Key changes:
- [specific transformation 1]
- [specific transformation 2]

Preserve: [what must stay the same — key facts, proper nouns, etc.]

<original>
[content to rewrite]
</original>
```

## Template 5: Code Generation Prompt

```
Write [language] code that:

Functionality: [what the code should do]
Input: [expected input format/type]
Output: [expected output format/type]

Requirements:
- [specific library/framework]
- [error handling expectations]
- [performance requirements]

Style:
- [naming conventions]
- [comment density]
- [typing/annotations]

Include: [tests / docstrings / examples / none]
```

## Template 6: Iterative Refinement Prompt

Use when building prompts through conversation:

```
I'm building a prompt for [use case]. Here's my current version:

<current_prompt>
[paste prompt]
</current_prompt>

The prompt works well for [what's working], but it fails when [specific failure cases].

Please:
1. Diagnose why it fails for those cases
2. Suggest 2-3 specific edits (show exact before/after text)
3. Add any missing guardrails you notice
4. Rate the prompt 1-10 and explain what would make it a 10
```
