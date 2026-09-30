---
name: prompt-engineering
description: Write, improve, debug and migrate prompts using the target model's own vendor guidance — Claude (Fable, Opus, Sonnet, Haiku), OpenAI GPT-6 and reasoning models, Google Gemini 3.x. Use for any prompt, system prompt, template, few-shot, or "make the model do X" request.
---

# Model-Aware Prompt Engineering

Prompting advice is model-specific. What helps one model (e.g. "think step by step", aggressive CAPS, prefills, low temperature) can hurt another. This skill routes every prompt task to the best practices for the **exact model** the prompt will run on.

Guidance below was compiled from official vendor docs (Sep 2026). Vendors update these pages often: when the stakes are high (production system prompt, migration, client deliverable), re-fetch the linked source for the target model and prefer it over this file if they disagree.

## Step 0 — Identify the target model (always first)

1. Find the model: the user names it, it's in their code (`model=`), or ask once. If unknown and the user can't say, write a vendor-neutral prompt (Universal Principles) and flag the 2–3 model-sensitive choices.
2. Identify the surface: raw API, Claude.ai / ChatGPT / Gemini app, Claude Code / agent harness, or an eval pipeline. API-only advice (effort, thinking params, prefill) doesn't apply in chat apps.
3. Identify the mode: single-turn chat, long-document analysis, tool-using agent, unattended/autonomous run, or multi-agent.
4. Jump to the model section below. Apply Universal Principles + that section. Mention which model's guidance you applied.

## Core workflow

1. **Clarify the goal** — what should the model do, for whom, and what does success look like? (Anthropic: have success criteria and a way to test before optimizing.)
2. **Pick the model + settings** — effort / reasoning level / thinking level first; it is a stronger, more reliable lever than prompt wording for depth vs. speed.
3. **Draft** — Universal Principles + model section.
4. **Strip** — remove legacy scaffolding the target model no longer needs (each section has a "Remove" list).
5. **Test and iterate** — run on representative inputs incl. edge cases; change one thing at a time.

## Adapting to skill level

- **Beginner** ("how do I get it to…"): write the prompt for them, explain 1–2 techniques with examples.
- **Intermediate** (has a draft): diagnose, then give specific edits with the *why*.
- **Advanced** (evals, production, migration): be concise; focus on parameters, removals, edge cases, eval design.

---

## Universal principles (all vendors)

1. **Be clear and direct.** Golden rule: would a smart colleague with no context understand and follow it? If not, neither will the model.
2. **Give context and the why.** "Your output is read aloud by TTS, so never use ellipses" beats "NEVER use ellipses".
3. **Say what to do, not what not to do.** "Write in flowing prose paragraphs" beats "Don't use markdown".
4. **Structure with delimiters.** XML tags for content boundaries (`<documents>`, `<example>`, `<context>`), Markdown headers for hierarchy. Be consistent in naming.
5. **Long context: data first, question last.** All three vendors recommend placing large documents at the top and the specific instruction/question at the end.
6. **Specify output format and scope explicitly** — schema, length, audience. Newer models follow literally; state when an instruction applies to *every* item.
7. **Examples:** 3–5 diverse, wrapped in tags, for non-reasoning or format-critical tasks. For reasoning models, try zero-shot first.
8. **Handle uncertainty:** tell the model what to do when unsure or when information is missing.
9. **Prompt as code:** version prompts, test them with evals, pin model snapshots in production.
10. **Match prompt style to desired output.** Markdown-heavy prompts produce markdown-heavy answers.

### Cross-model differences cheat-sheet

| Topic | Claude (5.x family) | OpenAI GPT-6 | Gemini 3.x |
|---|---|---|---|
| Depth lever | `effort` (low→max); thinking always on for Fable/Opus 5.5 | `reasoning.effort` (low, **medium default**, high, xhigh, max) | `thinking_level` (minimal, low, medium, high) |
| Default depth | Opus 5.5: **medium**; Sonnet 5.5 / Fable 5.1: **high** | medium | 3.5+ Flash: **medium** |
| Sampling params | Sonnet 5: non-default temperature/top_p/top_k → 400 error | temperature/top_p unsupported when reasoning ≠ none | Keep at defaults (temperature 1.0); lowering can cause looping |
| "Think step by step" | Remove; asking to reproduce reasoning can trigger `reasoning_extraction` refusals | Unnecessary for reasoning models | Optional nudge ("Think very hard") costs thinking tokens |
| Prefill | Not supported on Claude 4.6+ (400 error) | n/a | n/a |
| Instruction slot | `system` prompt | `developer` message / `instructions` param (per-request only!) | `system_instruction` |
| Structure | XML tags strongly preferred | Markdown + XML; Identity → Instructions → Examples → Context | XML or Markdown, consistent; `<role>`, `<instructions>`, `<constraints>` |
| Default verbosity | Opus 5.5 / Sonnet: calibrated; Opus 5: long | Favors lists/tables/markdown | Terse, direct |

---

## Anthropic Claude

**Sources:** [Overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) · [Best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) · [Models overview](https://platform.claude.com/docs/en/models/overview) · [Migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide) · [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) · [Thinking](https://platform.claude.com/docs/en/build-with-claude/thinking)

### Current lineup (Sep 2026)

| Model | ID | Use for |
|---|---|---|
| Fable 5.1 (and Mythos 5.1) | `claude-fable-5-1` | Hardest reasoning, long-horizon autonomous work |
| Opus 5.5 | `claude-opus-5-5` | Default: long-running agentic coding and knowledge work |
| Sonnet 5.5 | `claude-sonnet-5-5` | Speed + intelligence balance |
| Haiku 4.5 | `claude-haiku-4-5-20251001` | Fastest; extended thinking with `budget_tokens` |

### Claude-wide best practices (all current models)

- **XML tags** for everything mixed: `<documents><document index="1"><source/><document_content/></document></documents>`, `<example>`, `<instructions>`.
- **Long context (20k+ tokens):** documents at top, query at end (up to ~30% quality gain). Ask Claude to quote relevant passages first, then answer.
- **Role** in the system prompt focuses tone and domain.
- **Tone down aggressive language.** Since Opus 4.5, "CRITICAL: You MUST…" causes overtriggering. Write "Use this tool when…".
- **Action vs. suggestion is literal.** "Can you suggest changes?" → suggestions. "Change this function" → edits. Set a default with `<default_to_action>` or `<do_not_act_before_instructions>` blocks.
- **Parallel tool calls:** "If you intend to call multiple tools and there are no dependencies between them, make all independent calls in parallel… Never use placeholders or guess missing parameters."
- **No prefill** on the last assistant turn (Claude 4.6+ returns 400). Use format instructions, structured outputs, or XML-tag output instead.
- **Thinking:** Claude 4.6+ uses adaptive thinking (`thinking: {type: "adaptive"}` + `effort`); Fable/Mythos 5.x and Opus 5.5 have thinking always on. Prefer general instructions ("think thoroughly") over hand-written step plans. Put `<thinking>` in few-shot examples to shape reasoning style.
- **Append-only history** (Fable 5.1, Opus 5.5, Sonnet 5.5): pass assistant turns back unchanged incl. thinking blocks; never edit earlier turns — it invalidates later thinking. Use turn-scoped system messages (`clear_at: "next_user_message"`) for per-turn reminders.
- **Don't ask Claude to "show/explain your reasoning" in the response** on 5.x models → `reasoning_extraction` refusals. Use `thinking.display: "summarized"` instead.
- **`max_tokens`:** thinking counts toward it. Use up to 128k for long agentic turns on 5.x.
- **Formatting control:** match prompt style; use an `<avoid_excessive_markdown_and_bullet_points>` block for prose; ask for plain text explicitly if you don't want LaTeX math.
- **Overeagerness / over-engineering (coding):** "Only make changes that are directly requested or clearly necessary… Don't add features, refactor, or make 'improvements' beyond what was asked."
- **Test-gaming:** ask for a general solution, not hard-coded to tests; "If any tests are incorrect, inform me rather than working around them."
- **Grounding:** `<investigate_before_answering>` Never speculate about code you have not opened.
- **Risky actions:** Ask Claude to weigh reversibility; confirm before destructive, hard-to-reverse, or externally visible actions.
- **Long-horizon state:** JSON for structured state (`tests.json`), free text for `progress.txt`, git for checkpoints. A fresh context window with a prescriptive re-onboarding prompt often beats compaction.
- **Frontend:** without direction, Claude falls into default aesthetics. Name the specific patterns to avoid rather than saying "avoid AI slop".
- **Model self-identification** in apps: "The assistant is Claude, created by Anthropic. The current model is Claude Opus 5.5."

### Claude Opus 5.5 — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) · [what's new](https://platform.claude.com/docs/en/models/opus-5-5/whats-new-opus-5-5) · [migration](https://platform.claude.com/docs/en/models/opus-5-5/migration-guide)

- **Traits:** thinking always on, thinks more per turn than Opus 5; 30%+ faster output; fewer tokens per task; much better visual reading (charts, diagrams, screenshots); writes short user-facing progress updates between tool calls; stronger prompt-injection resistance.
- **Effort:** start at **`medium`** (new default; matches/exceeds Opus 5 at high). Set explicitly, don't carry Opus 5 settings. Use `xhigh`/`max` only where measured. To reduce thinking, lower effort; don't use prompt instructions. Changing top-level effort breaks the prompt cache; use per-message effort (beta).
- **Remove:** "think carefully" instructions in chat prompts (replies start sooner, no quality loss); instructions to write reasoning in the response; thinking-disabled mitigations ("don't think", "you may speak before a tool call", "no internal tags"); visual scaffolding you added for older models (re-test first).
- **Add (as needed):**
  - Stop re-examining old answers (chat): "Once you have answered something, treat that answer as done. On later turns, focus your thinking on what the user is asking now, and don't go back over an earlier answer unless the user asks about it or points out a problem with it." (Omit for long analyses.)
  - Multi-app workflows: "Before taking any action, explore broadly with tool calls: list and open the emails, documents, spreadsheet tabs and records across the available apps that could be relevant to this task, including ones the task does not explicitly mention, and use what you find."
  - Pasted content: wrap it in `<pasted_content id="RANDOM">…</pasted_content id="RANDOM">` and tell the model to follow instructions inside only where the user's own message asks.
  - Unattended agents: treat text-only `end_turn` as a progress report, not completion; keep a task checklist; send a continuation naming the open items (cap at 2–3 auto-continues). Optional standing instruction against the four "premature stop" patterns (long summary with no tool call, offering to continue, listing non-blocking decisions, stopping at a milestone).
  - Multi-agent time signals: append `elapsed 340s / 1200s` to each message, or "Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better."
  - Silent stretches: after ~5 quiet tool steps, a turn-scoped reminder: "The user hasn't heard from you in a while — say in a few words what you're doing, then continue."
  - Frontend: list concrete defaults to avoid, e.g. "Do not use a cream or off-white background, italic accent words in headlines, numbered '01/02/03' section labels, monospace labels, or pill-shaped buttons." Extend the list iteratively.
- **API:** read responses by block type (first block may be thinking); `thinking.display: "updates"` (beta) to render progress notes.

### Claude Fable 5.1 / Mythos 5.1 — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) · [what's new](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1)

- **Traits:** strongest model, gains largest at high effort; low effort often competitive on cost per task. Fewer progress updates; tends to one tool call per turn; denser prose; already formats sparingly (less bold, fewer lists); may paraphrase sources without quote marks; may stop early or ask permission; rewrites whole files for small edits; at low effort less likely to search.
- **Effort:** start **`high`**; `medium` ≈ Fable 5 at lower cost; `xhigh`/`max` can produce very long outputs.
- **Remove:** anti-formatting rules (it already formats less); "hold all findings for the final response"; "does this compile?" phrasing (safeguard false positives → ask "Are there any bugs?"); base64 blobs in tool output.
- **Add (as needed):**
  - Progress: "Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own…"
  - Autonomy: the "You are operating autonomously… check your last paragraph. If it is a plan, a question, or a promise about work you have not done, do that work now with tool calls" block, plus a "Delivering work" scope block (don't narrow/widen/swap scope; finish unblocked parts; suggestions go at the end).
  - Batching (per tool-result round as a turn-scoped system message): "First privately list what you need next; then request every item that doesn't depend on another's result in this one response."
  - Surgical edits: "When it will not affect the end result, try to surgically edit a file rather than rewrite the entire thing."
  - Writing: "Please remove all mannered prose." (say what you mean; use literal phrasing)
  - Low-effort search: search before answering when a name is unfamiliar or from a fast-moving area, including the name as written.
  - Quoting sources: give a full worked example of paraphrase + one short marked quote.
  - Long deliverables at xhigh/max: tell it reasoning + reply share one `max_tokens` limit; don't draft the whole output twice.

### Claude Sonnet 5.5 — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5) · [migration](https://platform.claude.com/docs/en/models/sonnet-5-5/migration-guide)

- **Traits:** at low/medium effort may check in too early or report code "done" without running tests; at high effort adds unrequested tests/docs/files; may answer from training instead of searching; may misread genuine mid-turn user messages as injections.
- **Effort:** default **`high`**; well-specified agentic coding `medium`, harder coding `high`, chat/latency `medium` or `low`. `xhigh`/`max` don't accept `thinking: {type: "between_tools"}`. `max_tokens` 128k for agentic work.
- **Remove:** "only use tools when strictly necessary" / "minimize tool calls"; "include your reasoning in the response"; "think less" (lower effort instead).
- **Add (as needed):**
  - Finish the job (low/medium): "Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step."
  - No extras: "When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it."
  - Limit xhigh/max thoroughness: no extra review/hardening rounds or reviewer subagents unless asked.
  - Ideas-only requests: "When the user asks for ideas, options or a plan, give them that and stop."
  - Search: "Use the search tool to check specifics that may have changed since your training… even when you feel confident."
  - Code verification: run a real check (tests, type-checker, build) before reporting done; say which check wasn't run and why.
  - Reasoning tasks with JSON output: "Think the problem through before you answer."

### Claude Opus 5 (previous gen) — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)

- **Traits:** longer default responses and narration; self-verifies unprompted (over-verifies if told to); expands scope; delegates to subagents readily; narrates self-corrections.
- **Effort:** default `high`; use `low`/`medium` liberally where quality holds; `xhigh` for hard coding/agentic work.
- **Remove:** "double-check / re-verify / use a subagent to verify" instructions; "don't think" rules.
- **Add:** conciseness ("Keep responses focused, brief, and concise…" + a short `<tone_preference>` reminder in long prompts); deliverable length ("cover the substance, but do not pad"); scope ("Deliver what was asked, at the scope intended…"); subagent limits; "Only correct an earlier statement when the error would change the user's code, conclusions, or decisions."

### Claude Sonnet 5 (previous gen) — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)

- **Traits:** calibrated length; **literal** instruction following (state scope: "Apply this formatting to every section, not just the first one"); more agentic; adaptive thinking on by default; good progress updates.
- **Effort:** default `high`; `xhigh` hardest coding; `medium` ≈ Sonnet 4.6 at high.
- **Remove:** non-default `temperature`/`top_p`/`top_k` (400 error); manual `budget_tokens` (400 error); "summarize after every 3 tool calls" scaffolding. New tokenizer ≈ +30% tokens vs 4.6, so raise `max_tokens`.
- **Add:** code review high-recall prompt ("Report every issue you find, including ones you are uncertain about… include your confidence level and an estimated severity"); concrete `<frontend_aesthetics>` constraints; warm tone line if desired.

### Claude Fable 5 / Mythos 5 — [guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)

- For hours-to-weeks autonomous work. Can over-plan, over-elaborate, occasionally fabricate progress on long runs, take unrequested "helpful" actions, and rarely call a send-to-user tool unless told.
- **Remove:** overly prescriptive step-by-step instructions (degrade quality); "show your thinking".
- **Add:** "When you have enough information to act, act…"; "Before reporting progress, audit each claim against a tool result from this session…"; "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits."; a `send_to_user` tool + instruction to use it for verbatim content; memory-file conventions; context on *why* the task matters.

### Claude Opus 4.8 and earlier 4.x — [Opus 4.8 guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8)

- Opus 4.8: thinking **off** unless `thinking: {type: "adaptive"}`; literal instruction following; favors reasoning over tools (describe why/how to use tools); fewer subagents; `xhigh` recommended for coding/agentic, start `max_tokens` ~64k at xhigh/max; defaults to cream/serif designs.
- Opus 4.5/4.6: over-responsive to aggressive system prompts; over-engineer; Opus 4.6 over-explores and over-spawns subagents. Opus 4.5 is sensitive to the word "think" when thinking is disabled ("consider", "evaluate" instead).
- Sonnet 4.5/4.6, Haiku 4.5: have **context awareness** (track remaining token budget). For agent harnesses with compaction: "Your context window will be automatically compacted… do not stop tasks early due to token budget concerns."
- **Haiku 4.5** has no dedicated guide: use the Claude-wide best practices; extended thinking via `thinking: {type: "enabled", budget_tokens: N}`; (general advice, not from a Haiku-specific doc: be more explicit and include examples, since it's the smallest model); knowledge cutoff Feb 2025, so supply current facts in context.

---

## OpenAI

**Sources:** [Prompting overview](https://developers.openai.com/api/docs/guides/prompting) · [Prompt engineering](https://developers.openai.com/api/docs/guides/prompt-engineering) · [Using GPT-6 (latest model)](https://developers.openai.com/api/docs/guides/latest-model) · [Reasoning best practices](https://developers.openai.com/api/docs/guides/reasoning-best-practices) · [Structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs) · Cookbook: [GPT-5](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide), [GPT-5.1](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide), [GPT-5.2](https://cookbook.openai.com/examples/gpt-5/gpt-5-2_prompting_guide), [Prompt optimizer](https://cookbook.openai.com/examples/gpt-5/prompt-optimization-cookbook)

### Lineup (Sep 2026)

| Model | ID | Use for |
|---|---|---|
| GPT-6 Astra | `gpt-6-astra` | Most demanding reasoning, coding, professional work |
| GPT-6.1 Sol | `gpt-6.1-sol` | Near-Astra at lower cost |
| GPT-6 Luna | `gpt-6-luna` | Fast, high-volume, cost-sensitive |
| o-series (o3, o4-mini) | legacy reasoning | Older reasoning-model guidance below |

### OpenAI-wide best practices

- **Roles:** `developer` > `user` > `assistant`. Developer message = function definition (rules, business logic); user message = arguments.
- **`instructions` param applies only to the current request.** With `previous_response_id`, earlier instructions are *not* carried over; resend them each turn.
- **Developer message order:** Identity → Instructions (rules, do/don't, tool-calling guidance) → Examples → Context.
- Markdown for hierarchy, XML tags for content boundaries. Few-shot examples in a compact, scannable block.
- **Caching:** stable, reusable content at the start of the prompt.
- **Prompts in code, not `v1/prompts`**: reusable prompt objects shut down Nov 30, 2026. Keep typed prompt builders in versioned modules, pass directly to the Responses API.
- Pin model snapshots in production.

### GPT-6 (Astra / Sol / Luna)

- **Settings:** `reasoning.effort` = low, **medium (default)**, high, xhigh, max. Astra and Sol don't support `none` (use `low`). Migration: keep your current effort, then test one level lower. Remove `temperature`, `top_p`, `top_logprobs` when reasoning is on. Responses API required for tool calling on Astra/Sol. Change effort mid-conversation with `configuration_update` items (keeps cache).
- **Traits:** Astra asks for clarification more readily; follows skill files very strongly; favors lists/tables/Markdown; uses stock phrases.
- **Add (as needed):**
  - Bias to action: "You should infer the user's intent and task scope from the instructions and prior conversation context. Your job is to bias towards action and carry the user's intended task to completion." Treat "can you…" / "help me…" as requests to do the work.
  - "Before asking the user clarifying questions, complete the work that is already authorized from context… The user should be approving a concrete, reviewable result."
  - Precedence: "The user's instructions take precedence over guidelines provided in a skill."
  - Prose: "Default to using clear, concise paragraphs, each developing one main idea. Use lists only when the information is genuinely parallel, sequential, or easier to compare."
  - Slop filter: avoid "Bottom Line:", "delve", "foster", "leverage"; state the action directly.
  - Delegation: parallelize via subagents when it saves time or improves quality; keep inter-agent messages human-legible.
  - Testing scope: don't write tests for reversible low-impact changes that mirror the implementation.
- **Remove:** approval-blocking language that causes needless pauses; generic "be concise" (already improved); redundant autonomy disclaimers; skill-file guidance that conflicts with user intent.
- **Agentic (GPT family):** plan thoroughly, give preambles before major tool calls, use a TODO tool; frontend defaults Tailwind + shadcn/ui + Lucide + Motion.

### OpenAI reasoning models (o-series; also applies to GPT-6 in spirit)

- Treat them like a senior colleague: give the **goal and constraints**, not steps.
- Keep prompts brief and direct. **Don't** add "think step by step" / "explain your reasoning".
- Try **zero-shot first**; add examples only if needed.
- Use delimiters and explicit success criteria / constraints.
- Markdown is off by default in API output for o-series; put `Formatting re-enabled` on the first line of the developer message to turn it on.
- Pass reasoning items back (Responses API, `store: true`) to avoid re-reasoning.

### Older GPT (GPT-4.1 / 4o / 5.x)

- Non-reasoning GPT models need **more explicit, step-by-step instructions** and benefit more from few-shot examples.
- GPT-5.x: see the cookbook guides linked above for agentic eagerness control, `verbosity`, and tool preambles.

---

## Google Gemini

**Sources:** [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) · [Gemini 3 developer guide](https://ai.google.dev/gemini-api/docs/gemini-3) · [What's new in Gemini 3.5](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) · [Models](https://ai.google.dev/gemini-api/docs/models) · [Thinking](https://ai.google.dev/gemini-api/docs/thinking) · [Vertex prompt strategies](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/prompts/prompt-design-strategies)

### Lineup (Sep 2026)

`gemini-3.8-flash` (newest flagship; long-horizon SWE, agents) · `gemini-3.5-flash` · `gemini-3.5-flash-lite` / `gemini-3.1-flash-lite` (cost) · `gemini-3.1-pro-preview` (complex reasoning; `-customtools` variant if it ignores tools) · Gemini 3.8 Live (voice).

### Gemini 3.x best practices

- **Settings:** keep `temperature`, `top_p`, `top_k` at defaults; lowering temperature can cause looping or degraded output. Use `thinking_level` (minimal / low / medium / high), not numeric `thinking_budget`. Gemini 3.5+ Flash defaults to **medium**; Gemini 3 / 3.1 default high. Remove `candidate_count`.
- **Be concise and direct.** Verbose, elaborate prompt engineering built for older models makes Gemini 3.x over-analyze. For hard tasks, raise `thinking_level` and simplify the prompt.
- **Verbosity:** defaults to terse, direct answers. Ask explicitly for a conversational or detailed style.
- **Structure consistently:** XML (`<role>`, `<instructions>`, `<constraints>`, `<context>`, `<task>`) or Markdown headings, not mixed.
- **Order:** role, behavioral constraints and format requirements in the system instruction (or top); large context next; the specific question **last**, with an anchor like "Based on the information above…".
- **Few-shot strongly recommended** for format and scope (Google: few-shot prompts significantly outperform zero-shot); keep example formats consistent.
- **Multimodal:** treat images/audio/video as first-class inputs and reference them explicitly. Try `media_resolution_high` for dense documents.
- **Time and grounding clauses** (esp. Flash): state the current year and the knowledge cutoff; for RAG: "You are a strictly grounded assistant limited to the information provided in the User Context."
- **Tools:** enable Google Search grounding for recent/obscure facts, Code Execution for arithmetic and counting. To cut excess tool calls: lower `thinking_level` or add "You have a limited action budget of <n> tool calls. Use them efficiently."
- **Function calling:** return exactly one `FunctionResponse` per `FunctionCall` with matching `id`/`name`; put multimodal content inside the response; keep thought signatures in history (GenerateContent API).
- **Agents:** Google publishes a research-evaluated agentic system instruction (plan dependencies and constraints, risk-assess reads vs. writes, abductive hypothesis testing, persistence, grounding) in the prompt-design-strategies page. Adapt it rather than inventing one.
- **Template:**
  ```
  <role>You are a specialized assistant for [domain]. You are precise, analytical, and persistent.</role>
  <instructions>1. Plan  2. Execute  3. Validate against the task  4. Format as requested</instructions>
  <constraints>Verbosity: [Low/Medium/High] · Tone: [Formal/Casual/Technical]</constraints>
  <context>[large data here]</context>
  <task>Based on the information above, [question].</task>
  ```

---

## Migration playbook (moving a prompt between models)

1. Read the target model's "Remove" list and delete legacy scaffolding first: CoT instructions, CAPS emphasis, anti-laziness pushes, verification boilerplate, prefills, sampling params.
2. Translate settings: Claude `effort` ↔ OpenAI `reasoning.effort` ↔ Gemini `thinking_level`. Start at the vendor's recommended default, not your old value.
3. Re-home instructions: Claude `system` / OpenAI `developer` (resend each turn if using `instructions` + `previous_response_id`) / Gemini `system_instruction`.
4. Re-check formatting defaults: Fable 5.1 formats less, GPT-6 more, Gemini terse.
5. Re-check agent behavior: autonomy (stops early vs. overreaches), tool-call eagerness, subagent spawning, progress updates.
6. Run the same eval set on old vs. new before switching.

## Prompt review checklist

1. Target model and settings stated (effort / reasoning / thinking level).
2. Clear task: would a context-free colleague understand it?
3. Context and "why" given for non-obvious rules.
4. Output format and scope explicit (incl. "every item").
5. Examples where format matters (zero-shot first for reasoning models).
6. Edge cases covered (empty, unexpected, ambiguous input) and uncertainty/missing-info handling defined.
7. No contradictions; no instructions that conflict with the model's defaults (see "Remove" lists).
8. No legacy scaffolding for this model (CoT, CAPS, prefill, sampling params, over-verification).
9. Long context: data first, question last.
10. Agentic: action vs. suggestion default, risky-action confirmation, stop/continue rules, progress-update expectations.
11. Testable against success criteria.

## Common failure modes

| Symptom | Likely cause | Fix (check model section) |
|---|---|---|
| Stops early / asks "Shall I…?" | Model's autonomy default (Fable 5.1, Sonnet 5.5 low effort, GPT-6 Astra) | Add autonomy + finish-the-job block; raise effort |
| Does more than asked | Opus 5 / Sonnet 5.5 high effort / Opus 4.x overeagerness | Scope block; "mention it at the end instead of doing it" |
| Overtriggers tools | Aggressive "MUST use" language (Claude 4.5+) | Calm "Use X when…" |
| Answers from stale memory | Low effort; "minimize tools" language | Search-verification nudge; remove tool-discouraging lines |
| Refusal `reasoning_extraction` | Prompt asks to reproduce reasoning (Claude 5.x) | Remove; use summarized thinking display |
| 400 / unsupported-parameter error | Prefill (Claude 4.6+), sampling params (Sonnet 5; GPT-6 with reasoning on), `budget_tokens` (Sonnet 5) | Remove the parameter |
| Looping / degraded (Gemini) | Lowered temperature | Reset to default 1.0 |
| Too verbose | Opus 5 default; GPT-6 lists | Model-specific conciseness snippet |
| Too terse | Gemini 3.x, Fable 5.1 density | Ask explicitly for detail or plainer prose |
| Ignores formatting on later items | Literal following (Sonnet 5, Opus 4.8) | "Apply to every section, not just the first" |
| Generic design | Default aesthetics | Name concrete styles to avoid |
| Inconsistent output format | No format specified | Explicit JSON schema / template or structured outputs |
| Hallucinated facts | No uncertainty clause, no grounding | "If unsure, say so"; quote sources first; enable search |
| Generic / shallow output | Vague task | Add audience, depth, domain, examples, the why |
| Fails on edge cases | No edge-case or negative examples | Add them to examples and tests |
| Prompt too long / expensive | Over-specified for a capable model | Cut what the model already does by default; test minimal version |

## Reference files (bundled)

These older references still apply for general technique. Where they conflict with the model sections above (e.g. they recommend prefilling, manual CoT, or `budget_tokens`), the model sections win.

- Techniques (XML, CoT, few-shot, chaining): [references/claude-techniques.md](references/claude-techniques.md)
- System prompt patterns: [references/system-prompt-patterns.md](references/system-prompt-patterns.md)
- Templates: [references/templates.md](references/templates.md)
- Hallucination reduction: [references/hallucination-reduction.md](references/hallucination-reduction.md)
- Production optimization (caching, batch, latency): [references/production-optimization.md](references/production-optimization.md)
- Evals & testing: [references/evals-and-testing.md](references/evals-and-testing.md)
- Anthropic Console tools: [references/console-tools.md](references/console-tools.md)

## Output format when delivering a prompt

1. **Target:** model + recommended settings (effort/thinking level, max tokens, sampling params to omit).
2. **The prompt** in a code block (system/developer + user template, with `{{variables}}`).
3. **Why it's built this way:** 3–5 bullets tying choices to that model's guidance.
4. **Removed from your original** (if revising) and why.
5. **How to test:** 3–5 test inputs incl. an edge case.
