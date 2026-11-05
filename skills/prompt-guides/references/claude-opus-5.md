# Prompting Claude Opus 5 (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
- Fetched: 2026-11-05. Append `.md` to the URL for the raw Markdown source.
- Covers: the prompting patterns specific to Claude Opus 5. The page links to the Claude Opus 5 overview for the model's specifications, and to the best-practices page for techniques that apply across all current Claude models.

Claude Opus 5 is built for complex agentic coding and enterprise work, with particular strengths in long-horizon agentic tasks. It performs well out of the box on existing Claude Opus 4.8 prompts.

## Capability improvements

- **Agentic coding:** strongest on difficult tasks (multi-file features, larger refactors, end-to-end work). Completes full tasks rather than leaving stubs, and performs best given the complete task specification up front and left to run. It also performs well on easier tasks such as single-turn edits, where the difference from prior models is smaller.
- **Code review and bug-finding:** high precision and recall: it finds real bugs at a high rate per pass, and its additional findings are mostly real issues rather than false positives. Accuracy holds at lower effort, which supports a fast pass at review time and a more thorough pass later. If your review prompt says "only report high-severity issues" or "be conservative," the model follows it literally and reports less; ask it to report everything and filter in a separate pass instead.
- **Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency. Start at the default (`high`), then use `low`/`medium` liberally as the primary control for cost and latency wherever quality holds; step up to `xhigh` for demanding coding and agentic work. Re-run an effort sweep if you carried defaults over from a prior model.
- **Vision:** strong on charts, documents, diagrams, and UI replication. Re-validate prompt-side vision workarounds tuned for prior models. Vision is strongest with tools to iteratively analyze, crop, and visually verify; tool use is a more cost-effective lever than thinking alone.
- **Long context:** 1M-token window as both default and maximum; instruction following, tool calling, and reasoning stay consistent throughout.
- **Office/document tasks:** complex multi-sheet spreadsheets and well-structured slide decks; prompt with any specific styles or templates.
- **Multi-agent coordination:** coordinates subagent teams well (writer-verifier patterns, little overwriting). For cost-sensitive workloads, cap delegation (below).

## Response length and verbosity

Default user-facing responses run longer than prior Opus models'. Effort controls how much the model thinks, not how much it says: lowering effort does not reliably shorten visible responses. Prompt for length explicitly:

> Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and spend most of the response on the main answer. When asked to explain something, give a high-level summary unless an in-depth explanation is specifically requested.

In a long system prompt, pair that with a short reminder near the end: `<tone_preference>Keep outputs reasonably concise.</tone_preference>`

## User-facing progress updates

The model narrates readily during agentic work: it announces what it is about to do, and per-message output runs long. Describe the cadence and shape you want:

> Before your first tool call, say in one sentence what you're about to do. While working, give a brief update only when you find something important or change direction. When you finish, lead with the outcome: your first sentence should answer "what happened" or "what did you find," with supporting detail after it for readers who want it.

To tune narration up or restyle it, use the same lever in the other direction: positive examples of the style you want beat instructions about what not to do.

## Written deliverable length

Files written to disk (reports, Markdown documents, summaries) also run longer than prior models. Calibrate explicitly:

> Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate.

## Task scope and over-verification

Opus 5 verifies its own work without being told to. **Remove** explicit verification instructions ("include a final verification step," "use a subagent to verify") and legacy verification scaffolding: they cause over-verification, and removing them cuts tokens with no quality loss.

The model can also expand task scope, adding unrequested steps. For narrow tasks:

> Deliver what was asked, at the scope intended. Make routine judgment calls yourself, and check in only when different readings of the request would lead to materially different work. If the request seems mistaken or a better approach exists, say so in a sentence and continue with the task as asked rather than quietly narrowing, widening, or transforming it. Finish the whole task, and stop short of actions that are clearly beyond what was asked.

## Controlling subagent spawning

Opus 5 delegates to subagents more readily than prior models. Delegation pays on genuinely independent, sizeable tracks; it multiplies cost on small tasks. Give explicit guidance or deterministic caps:

> Delegate to a subagent only for large tasks that are genuinely independent and parallelizable, such as a wide multi-file investigation. Do not delegate work you can finish yourself in a handful of tool calls, and do not use subagents to verify or double-check your own work. If one subagent can complete the task, use one rather than several, and keep spawn counts low.

If the harness is Claude Code or the Claude Agent SDK, the deterministic caps are the `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` and `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` environment variables and the SDK's `max_budget_usd` (TypeScript: `maxBudgetUsd`) option. Older versions of Claude Code don't have them, so update Claude Code and any pinned SDK before pointing your harness at Opus 5. Claude Code adds a delegation instruction of its own on Opus 5 only under its `claude_code` system prompt preset; with a custom or omitted system prompt, add one yourself, such as the sample above.

## Self-correction

The model catches and fixes its own mistakes well unprompted. Avoid instructing re-checks it already performs ("double-check your answer," "re-verify before responding"); they compound with its own behavior and add cost without improving results.

It also narrates corrections more than prior models. To limit that to corrections that matter:

> Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.

## Running with thinking disabled

Thinking is on by default and can be disabled only at effort `high` or below. A request that combines `thinking: {"type": "disabled"}` with effort `xhigh` or `max` returns a 400 error. With thinking disabled, two artifacts can appear; the primary mitigation for both is to keep thinking enabled and lower effort instead (thinking on at `low` effort usually beats thinking off at similar cost):

- **Tool calls as text:** the model occasionally writes a tool call into user-facing text instead of a structured `tool_use` block. The call never runs, and the leaked text pollutes later turns. Most common on tool-heavy workloads such as search.
- **Internal XML tags in output:** `<thinking>` or other internal tags can leak into visible responses. If your system prompt tells the model not to think or not to reason, remove that rule; it increases tag leakage.

If thinking must stay disabled, one combined instruction mitigates both:

> When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.

Instructions that call out thinking tags by name are less effective than this general form, so avoid naming them.

## Reasoning in the response

Prompts, skills, and tool descriptions that ask Opus 5 to write out its thinking or reasoning, verbatim or in a fixed format, might be declined with the `reasoning_extraction` refusal category. Ask instead for a short explanation of the answer or a summary of the actions taken. If you need the reasoning itself, keep thinking on and read the summarized thinking blocks (`display: "summarized"`).
