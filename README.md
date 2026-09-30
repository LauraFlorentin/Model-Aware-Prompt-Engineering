# Model-Aware Prompt Engineering

A Claude skill that writes, reviews, debugs and migrates prompts using the **official prompting guidance for the exact model the prompt will run on**. It covers Anthropic Claude, OpenAI GPT and Google Gemini.

Prompting advice doesn't carry over cleanly between models. "Think step by step", CAPS-lock emphasis, prefilled responses and a low temperature can help one model and break another. Before drafting anything, this skill works out the target model, its settings and how it will be used (chat, long documents, agent, unattended run). It then applies that vendor's own best practices, including what to *remove* from prompts written for older models.

## Coverage (as of September 2026)

| Vendor | Models with dedicated guidance |
|---|---|
| Anthropic | Opus 5.5, Fable 5.1 / Mythos 5.1, Sonnet 5.5, Opus 5, Sonnet 5, Fable 5 / Mythos 5, Opus 4.8 and earlier 4.x, Haiku 4.5 |
| OpenAI | GPT-6 Astra / GPT-6.1 Sol / GPT-6 Luna, o-series reasoning models, older GPT-4.1 / 4o / 5.x |
| Google | Gemini 3.x (3.8 Flash, 3.5 Flash / Flash-Lite, 3.1 Pro) |

The skill also includes:

- a comparison table of the three vendors: effort and thinking settings, sampling parameters, prefill support, where instructions go, and default verbosity
- a playbook for moving a prompt from one model to another
- a review checklist and a table of common failures with fixes
- a standard format for handing over a finished prompt: target settings, the prompt, why it's built that way, what was removed, and test inputs

## Repository layout

```
skills/prompt-engineering/
├── SKILL.md                 # Model router + per-model guidance
└── references/
    ├── claude-techniques.md
    ├── system-prompt-patterns.md
    ├── templates.md
    ├── hallucination-reduction.md
    ├── production-optimization.md
    ├── evals-and-testing.md
    └── console-tools.md
```

The reference files cover general techniques. Some of them were written before Claude 4.6 (for example, their prefill advice). Where they disagree with `SKILL.md`, the model sections in `SKILL.md` take precedence.

## Installation

**Claude Code:** copy the skill folder into your skills directory.

```bash
git clone https://github.com/LauraFlorentin/Model-Aware-Prompt-Engineering.git
cp -r Model-Aware-Prompt-Engineering/skills/prompt-engineering ~/.claude/skills/
```

**Claude.ai / Claude desktop:** zip the `skills/prompt-engineering` folder and upload it under Settings → Capabilities → Skills.

## Usage

Ask for a prompt and name the model:

- "Write a system prompt for a support agent running on Claude Sonnet 5.5"
- "Migrate this GPT-5 prompt to Opus 5.5"
- "Why does this prompt loop on Gemini 3.5 Flash?"

If you don't name a model, the skill asks for one once. If the model can't be pinned down, it writes a vendor-neutral prompt and flags the choices that depend on the model.

## Sources

All guidance is compiled from official vendor documentation:

- **Anthropic:** [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview), [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), [Prompting Claude Opus 5.5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) and the other per-model guides linked in `SKILL.md`
- **OpenAI:** [Prompting](https://developers.openai.com/api/docs/guides/prompting), [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering), [Latest model](https://developers.openai.com/api/docs/guides/latest-model), [Reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices), [Cookbook prompting guides](https://cookbook.openai.com/examples/gpt-5/gpt-5-2_prompting_guide)
- **Google:** [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies), [Gemini 3 developer guide](https://ai.google.dev/gemini-api/docs/gemini-3), [What's new in Gemini 3.5](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5)

Vendors update these pages often. For production or other high-stakes prompts, the skill tells Claude to re-read the linked source for the target model and to trust the live page over the snapshot here.

## Maintaining

When a vendor ships a new model or prompting guide:

1. Add a section under the vendor in `SKILL.md`: traits, effort default, what to remove, what to add, and links to the sources.
2. Update the comparison table and the lineup table.
3. Update the "as of" date in this README.

## License

MIT, see [LICENSE](LICENSE).
