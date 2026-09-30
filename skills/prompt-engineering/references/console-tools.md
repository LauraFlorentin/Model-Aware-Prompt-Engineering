# Anthropic Console Tools

## Prompt Generator

The Anthropic Console includes a prompt generator that creates a first-draft prompt from a plain-language description. Access it at: https://console.anthropic.com

**When to use:** Starting from scratch with no prompt draft. Describe your task in plain language and the generator produces a structured prompt with system message, user message, and examples.

**When to skip:** If you already have a working prompt or need fine-grained control.

**Workflow:**
1. Describe your task in the generator
2. Review the generated prompt — treat it as a starting point, not final
3. Customize: add domain-specific examples, adjust constraints, tune output format
4. Test against your eval set

## Prompt Improver

Analyzes an existing prompt and suggests improvements. Useful for catching common issues you might miss.

**When to use:** You have a prompt that mostly works but underperforms on some cases.

**What it checks for:**
- Ambiguous instructions
- Missing output format specifications
- Opportunities for few-shot examples
- Structural improvements (XML tags, section ordering)

## Prompt Templates & Variables

For production prompts with dynamic inputs, use template variables:

```
Analyze the following {{content_type}} and provide a {{analysis_depth}} analysis.

Focus on: {{focus_areas}}

<document>
{{document_content}}
</document>
```

**Benefits:**
- Separate prompt logic from data
- Version control the template independently
- Test with different variable values
- Reuse across similar tasks

**Best practice:** Keep static instructions in the template, put only truly variable content in `{{variables}}`.

## Structured Outputs

Force Claude to return valid JSON matching a specific schema. This is an API feature, not a prompting technique — it guarantees schema compliance at the model level.

```python
response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Classify this email..."}],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "email_classification",
            "schema": {
                "type": "object",
                "properties": {
                    "category": {
                        "type": "string",
                        "enum": ["billing", "technical", "account", "feedback", "other"]
                    },
                    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
                    "requires_human": {"type": "boolean"}
                },
                "required": ["category", "confidence", "requires_human"]
            }
        }
    }
)
```

**When to use structured outputs vs. prompting for JSON:**
- Use structured outputs when you MUST have valid JSON (production APIs, data pipelines)
- Use prompt-based JSON when you need flexibility or the output schema varies by input

## Evaluation Tool

The Console's eval tool lets you:
- Define test cases with expected outputs
- Run prompts against test cases
- Compare prompt versions side-by-side
- Track scores over time

**Workflow:**
1. Create a test set in the Console (or upload JSON)
2. Assign your prompt to the test set
3. Run evaluation
4. Review per-case results and aggregate scores
5. Make prompt changes and re-run to compare

**Tip:** Export results for deeper analysis or to share with your team.

## Workbench

The Console Workbench is an interactive prompt playground:
- Test prompts in real-time
- Adjust model, temperature, max_tokens
- Compare system prompt vs. user message placement
- View token counts and estimated costs

**Best for:** Rapid iteration on prompt wording before committing to code.
