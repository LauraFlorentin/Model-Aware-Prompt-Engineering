# System Prompt Patterns for Claude

## The Contract Pattern (Recommended Default)

Structure system prompts like a clear contract. This is the most reliable pattern for Claude 4.x models.

```
You are: [role — one line]

Goal: [what success looks like]

Constraints:
- [constraint 1]
- [constraint 2]
- [constraint 3]

If unsure: [how to handle uncertainty]

Output format: [expected structure]
```

### Example: Customer Support Bot

```
You are: A friendly customer support agent for Acme SaaS (project management tool).

Goal: Resolve customer issues quickly while maintaining a warm, professional tone. Aim for first-contact resolution.

Constraints:
- Never share internal pricing logic or discount thresholds
- Never promise features on the roadmap — say "I'll pass that feedback to our team"
- Escalate to human agent if: billing disputes > $500, account security issues, legal threats
- Always verify the customer's account before making changes

If unsure: Say "Let me check on that for you" and ask a clarifying question. Never guess.

Output format: Start with acknowledgment of the issue, then provide the solution or next step. End with "Is there anything else I can help with?"
```

## The Persona Pattern

For creative or conversational applications where personality matters.

```
<persona>
[Name/role with personality traits]
[Communication style: formal/casual/technical/etc.]
[Key behaviors and quirks]
</persona>

<knowledge>
[What this persona knows and doesn't know]
[Domain boundaries]
</knowledge>

<guardrails>
[What to never do or say]
[How to handle off-topic requests]
</guardrails>
```

## The Pipeline Pattern

For agents or multi-step workflows where Claude processes inputs through a defined pipeline.

```
You process incoming [inputs] through this pipeline:

1. CLASSIFY: Determine the [input] type → [category A / category B / category C]
2. VALIDATE: Check for [specific requirements]. If invalid, respond with [error format].
3. PROCESS: Based on classification:
   - Category A → [specific instructions]
   - Category B → [specific instructions]
   - Category C → [specific instructions]
4. FORMAT: Return results as [output format]

Always complete all steps. If a step fails, stop and report which step failed and why.
```

## The Evaluator Pattern

For prompts that review, score, or assess content.

```
Evaluate the following [content type] against these criteria:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| [Criterion 1] | [1-5] | [What excellent/poor looks like] |
| [Criterion 2] | [1-5] | [What excellent/poor looks like] |
| [Criterion 3] | [1-5] | [What excellent/poor looks like] |

For each criterion, provide:
- Score (1-10)
- One sentence justification
- One specific improvement suggestion

End with an overall weighted score and top 3 priority improvements.
```

## Anti-Patterns to Avoid

- **Overloaded system prompts:** If >2000 words, split into user-message context instead
- **Contradictory instructions:** Claude follows the last/most specific instruction — be consistent
- **Vague roles:** "Be helpful" adds nothing. Be specific about domain and behavior
- **Missing edge cases:** Always define what happens with unexpected/ambiguous input
- **No output format:** Without format guidance, Claude guesses — and may guess differently each time
