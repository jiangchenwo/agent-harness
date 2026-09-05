# Prompting Claude Opus 4.8 (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8
- Fetched: 2026-08-12. Append `.md` to the URL for the raw Markdown source.
- Covers: the prompting patterns specific to Claude Opus 4.8. The page defers the API changes of moving to Opus 5 to the migration guide, and techniques that apply across all current Claude models to the best-practices page.

Claude Opus 4.8 has particular strengths in long-horizon agentic work, knowledge work, vision, and memory tasks. It performs well out of the box on existing Claude Opus 4.7 prompts.

## Response length and verbosity

Opus 4.8 calibrates response length to how complex it judges the task, not a fixed verbosity: shorter on simple lookups, much longer on open-ended analysis. To decrease: "Provide concise, focused responses. Skip non-essential context, and keep examples minimal." Positive examples of the right concision level beat negative instructions.

## Calibrating effort and thinking depth

Start with `xhigh` effort for coding and agentic use cases; use a minimum of `high` for most intelligence-sensitive use cases. Effort matters more for this model than any prior Opus, so experiment actively when upgrading. Levels:

- `max`: gains in some use cases but diminishing returns and occasional overthinking; test for intelligence-demanding tasks.
- `xhigh`: best setting for most coding and agentic use cases.
- `high`: balances tokens and intelligence; the floor for intelligence-sensitive work.
- `medium`: cost-sensitive, trading off intelligence.
- `low`: short, scoped, latency-sensitive tasks that are not intelligence-sensitive.

Opus 4.8 respects effort strictly, especially at the low end: at `low`/`medium` it scopes work to what was asked. Under-thinking risk on moderately complex tasks at `low`. If you see shallow reasoning, raise effort rather than prompting around it; if `low` must stay: "This task involves multistep reasoning. Think carefully through the problem before responding."

**Thinking is off unless you explicitly set `thinking: {type: "adaptive"}`.** Triggering is steerable once enabled. To damp: "Thinking adds latency and should only be used when it will meaningfully improve answer quality — typically for problems that require multistep reasoning. When in doubt, respond directly."

At `max`/`xhigh` effort, set a large max output token budget so the model has room to think and act across subagents and tool calls. Start at 64k tokens and tune.

## Tool use triggering

Opus 4.8 tends to favor reasoning over tool calls, which produces better results in most cases. To increase tool usage: raise effort (`high`/`xhigh` show substantially more tool use in agentic search and coding), and explicitly describe when, why, and how to use each tool.

## User-facing progress updates

More regular, higher-quality updates through long agentic traces. Remove scaffolding that forces interim status messages ("After every 3 tool calls, summarize progress"). If updates are miscalibrated, describe what they should look like and give examples.

## More literal instruction following

Opus 4.8 interprets prompts literally and explicitly, particularly at lower effort. It does not silently generalize an instruction across items or infer unstated requests. Upside: precision and less thrash, which suits tuned API prompts, structured extraction, predictable pipelines. State scope explicitly when an instruction should apply broadly: "Apply this formatting to every section, not just the first one."

## Tone and writing style

Tends toward a direct, opinionated style with minimal validation-forward phrasing and sparing emoji use. Re-evaluate style prompts against the new baseline; for warmth: "Use a warm, collaborative tone. Acknowledge the user's framing before answering."

## Controlling subagent spawning

**Opus 4.8 tends to spawn fewer subagents by default.** The behavior is steerable; give explicit guidance on when subagents are desirable:

> Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see). Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.

## Design and frontend defaults

Opus 4.8 has a persistent default house style: warm cream/off-white backgrounds (~#F4F1EA), serif display type (Georgia, Fraunces, Playfair), italic word-accents, terracotta/amber accent. Reads well for editorial/hospitality/portfolio; feels off for dashboards, dev tools, fintech, healthcare, enterprise. Generic instructions ("don't use cream") shift it to a different fixed palette rather than producing variety. Two reliable approaches:

1. **Specify a concrete alternative.** The model follows explicit specs (palette hexes, radius, typography, structure, transitions) precisely.
2. **Have the model propose options before building:** "Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface — one-line rationale). Ask the user to pick one, then implement only that direction."

Opus 4.8 needs less anti-slop prompting than earlier models; this short snippet suffices alongside the variety approaches: `<frontend_aesthetics>NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.</frontend_aesthetics>`

## Interactive coding products

Opus 4.8 uses more tokens in interactive multi-turn settings, primarily because it reasons more after user turns (improving long-horizon coherence at token cost). To maximize performance and efficiency: `xhigh` or `high` effort, autonomous features like an auto mode, fewer required human interactions, and a well-specified first turn (task, intent, constraints upfront). Ambiguous prompts drip-fed over turns reduce token efficiency and sometimes performance.

## Code review harnesses

Meaningfully better at finding bugs than prior models (higher recall and precision internally), but a harness tuned for an earlier model may show lower recall. That is a harness effect. "Only report high-severity issues" / "be conservative" / "don't nitpick" are followed faithfully: same investigation depth, fewer reported findings. Recommended language:

> Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.

For single-pass self-filtering, set a concrete bar: "report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences." Validate on your evals.

## Computer use

Works across resolutions up to 2576px / 3.75MP. 1080p balances performance and cost; 720p or 1366×768 for cost-sensitive workloads. Test your own settings; effort tuning helps.
