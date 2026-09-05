# Prompting Claude Sonnet 5 (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5
- Fetched: 2026-09-05. Append `.md` to the URL for the raw Markdown source.
- Covers: the prompting patterns specific to Claude Sonnet 5. The page defers capabilities and API changes to the What's New page, and techniques that apply across all current Claude models to the best-practices page.

Claude Sonnet 5 has particular strengths in coding and agentic tasks and performs well out of the box on existing Claude Sonnet 4.6 prompts. API changes from 4.6: adaptive thinking on by default, sampling parameters not accepted, manual extended thinking removed, and a new tokenizer.

## Response length and verbosity

Sonnet 5 calibrates length to task complexity (shorter on simple lookups, longer on open-ended analysis) rather than a fixed verbosity. To decrease verbosity: "Provide concise, focused responses. Skip non-essential context, and keep examples minimal." Positive examples of the right level of concision beat instructions about what not to do.

## Calibrating effort and thinking depth

Effort defaults to `high` (same as Sonnet 4.6). Levels:

- `max`: absolute maximum capability, no constraints on token spending.
- `xhigh`: recommended for the hardest coding and agentic use cases.
- `high`: the default; balances tokens and intelligence for most use cases.
- `medium`: cost-sensitive use cases, trading off intelligence.
- `low`: short, scoped, latency-sensitive tasks that are not intelligence-sensitive.

Cross-model mapping when migrating: Sonnet 5 at `medium` ≈ Sonnet 4.6 at `high`; Sonnet 5 at `high` ≈ Sonnet 4.6 at `max`. When benchmarking, match by observed thinking length rather than effort name.

Sonnet 5 respects effort strictly, especially at the low end: at `low` and `medium` it scopes work to what was asked. On moderately complex tasks at `low` there is some risk of under-thinking. If you see shallow reasoning, raise effort rather than prompting around it; if `low` must stay for latency: "This task involves multistep reasoning. Think carefully through the problem before responding."

**Adaptive thinking is on by default**, a change from Sonnet 4.6, where requests without a `thinking` field ran without thinking. Turn it off with `thinking: {type: "disabled"}`. `max_tokens` is a hard limit on total output (thinking plus response text), so revisit it for workloads that ran thinking-off on 4.6. If you ran thinking off on 4.6, try thinking on at lower effort on Sonnet 5.

Thinking triggering is steerable. Too many thinking blocks (common with large system prompts): "Thinking adds latency and should only be used when it will meaningfully improve answer quality, typically for problems that require multistep reasoning. When in doubt, respond directly." Under-thinking at `medium` on hard workloads: raise effort first, prompt second.

**Manual extended thinking** (`thinking: {type: "enabled", budget_tokens: N}`) returns a 400 error on Sonnet 5. Use adaptive thinking with effort.

**Token budget warning:** at `high`/`xhigh`/`max`, leave headroom in `max_tokens`. Adaptive thinking can consume most of a tight budget and leave a truncated answer with `stop_reason: "max_tokens"`. Also, Sonnet 5's new tokenizer produces approximately 30% more tokens for the same text, so `max_tokens` limits tuned for Sonnet 4.6 may truncate equivalent output.

## Tool use triggering

Sonnet 5 is more agentic than 4.6 by default: it reaches for tools and runs self-verification loops more readily. With thinking disabled it is less likely to reach for tools, so add an explicit nudge if you rely on tool calls with thinking off. Higher effort (`high`/`xhigh`) substantially increases tool usage in agentic search and coding. For more tool use, explicitly describe when, why, and how to use each tool.

## User-facing progress updates

Sonnet 5 gives regular, higher-quality updates through long agentic traces. If you added scaffolding to force interim status messages ("After every 3 tool calls, summarize progress"), try removing it. If update length or content is miscalibrated for your use case, describe what updates should look like and give examples.

## More literal instruction following

Sonnet 5 interprets prompts literally and explicitly, particularly at lower effort. It does not silently generalize an instruction from one item to another or infer requests you didn't make. Upside: precision, which suits tuned API prompts, structured extraction, and predictable pipelines. If an instruction should apply broadly, state the scope: "Apply this formatting to every section, not just the first one."

## Tone and writing style

Prose style may shift from 4.6; re-evaluate style prompts against the new baseline (e.g. "Use a warm, collaborative tone. Acknowledge the user's framing before answering."). **Setting `temperature`, `top_p`, or `top_k` to a non-default value returns a 400 error**, which is new for Sonnet-class models. Remove these parameters when migrating; steer tone and variety through the system prompt.

## Design and frontend defaults

Sonnet 5 can settle into a consistent default visual style on open-ended briefs; generic instructions ("don't use that color," "make it clean and minimal") just shift it to a different fixed palette. Two reliable approaches:

1. **Specify a concrete alternative.** The model follows explicit specs precisely. The official example pins tonal palette (five hex values), corner radius, typography, section structure, and hover transitions for a landing page.
2. **Have the model propose options before building.** Since `temperature` is unavailable, this is the recommended way to get different design directions across runs: "Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction."

Short anti-slop directive for the system prompt: `<frontend_aesthetics>NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.</frontend_aesthetics>`

## Interactive coding products

For coding products, use `xhigh` or `high` effort, add autonomous features (auto mode), and reduce required human interactions. Specify task, intent, and constraints upfront in the first human turn: well-specified upfront prompts maximize autonomy and intelligence; ambiguous prompts drip-fed over multiple turns reduce token efficiency and sometimes performance.

## Code review harnesses

A harness tuned for an earlier model may show lower recall on Sonnet 5. That is a harness effect, not a capability regression. Prompts like "only report high-severity issues," "be conservative," "don't nitpick" are followed more faithfully: the model investigates just as deeply but reports fewer findings. Precision rises; measured recall can fall. Recommended language:

> Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.

Usable even without an actual second step. If you do want single-pass self-filtering, be concrete about the bar instead of qualitative terms: "report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences." Validate recall/F1 changes on your evals.

## Computer use

Sonnet 5 supports the `computer_toolset_20260801` toolset (Claude API and Google Cloud) and the earlier `computer_20251124` tool version. On the Claude API and Google Cloud it also supports the browser use tool (`browser_toolset_20260801`) for tasks inside webpages. Resolution up to 2576px / 3.75MP. Internal testing: 1080p balances performance and cost; 720p or 1366×768 are strong lower-cost options. Test your own settings; effort tuning helps here too.
