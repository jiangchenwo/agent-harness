# Prompting guidance for GPT-5.6 Sol (official guide, condensed)

- Source: https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6
- Fetched: 2026-09-05. Append `.md` to the URL for the raw Markdown source.
- Covers: adapting prompts, tool descriptions, agent instructions, or prompt stacks to GPT-5.6 Sol or the GPT-5.6 family. The page defers API details, limits, pricing, and feature availability to the GPT-5.6 model guide.

GPT-5.6 works best when prompts define the outcome, important constraints, available evidence, and completion bar, then leave room for the model to choose an efficient path. In OpenAI's internal coding-agent eval sample, leaner system prompts improved scores by roughly 10 to 15 percent while cutting total tokens by 41 to 66 percent and cost by 33 to 67 percent. Those numbers are directional; validate on your own tasks.

## Simplify prompts first

Start with a working prompt and tool set; remove one group of instructions, examples, or tools at a time, rerunning the same evals. Trim: repeated statements of the same rule; repeated style/process instructions that don't change behavior; examples that don't change behavior; process instructions for behavior the model already performs reliably; tools unrelated to the task. Keep: the user-visible outcome; success criteria and stopping conditions; safety, business, evidence, and permission constraints; context-dependent tool-routing rules; required output shape and validation. Then check the remainder for contradictions: GPT-5-class models follow prompt contracts closely, so conflicting rules create more instability than missing detail.

## Outcome-first prompts and stopping conditions

Describe the destination, not every step. Official example shape: "Resolve the customer's issue end to end. Success means: make the eligibility decision from available policy and account evidence; complete any allowed action before responding; return completed_actions, customer_message, and blockers; if required evidence is missing, ask for the smallest missing field."

Reserve ALWAYS/NEVER/must/only for true invariants (safety rules, required fields, actions that must never happen); use decision rules for judgment calls (when to search, ask, use a tool, keep iterating). Preserve explicit user values; when the correct value is implicit, give decision criteria rather than universal defaults, keyword maps, or broad semantic shortcuts.

Stopping-condition sample: "Resolve the request in the fewest useful tool loops, but do not let loop minimization outrank correctness, required evidence, calculations, or required citations. After each result, ask whether the core request can now be answered with useful evidence. If yes, answer. If required evidence is still missing, name the missing fact and use the smallest useful fallback."

## Personality, collaboration, and response length

GPT-5.6 is more concise by default than GPT-5.5, so broad brevity instructions ("Be concise") may now be unnecessary or make responses too brief; keep them only where they reliably help. For consistent control, set `text.verbosity` (`low`/`medium`/`high`) as the default detail level and use the prompt for task-specific requirements.

Define personality (tone, warmth, directness, formality, humor, empathy, polish) and collaboration style (when to ask questions, make assumptions, take initiative, explain tradeoffs, check work, handle uncertainty) separately, and keep both short; neither replaces goals, success criteria, tool rules, or stopping conditions.

For shorter answers, give a priority order rather than a length cap: "Lead with the conclusion. Include the evidence needed to support it, any material caveat, and the next action. Omit secondary detail and repetition. Keep all required facts, decisions, caveats, and next steps. Trim introductions, repetition, generic reassurance, and optional background first."

Broad labels ("friendly," "empathetic") are ambiguous, so describe the writing choices instead: "State the answer directly. If the user reports a problem, acknowledge the specific issue before giving the next step. Use reassurance only when it is relevant. Omit generic praise and unnecessary sign-offs." Avoid blanket language rules ("always respond in the user's language") unless truly required. For editing/rewriting/summaries: "Preserve the requested artifact, length, structure, genre, and factual claims first. Improve clarity, flow, and correctness without adding new claims, sections, or a more promotional tone unless requested."

## Define autonomy and approval boundaries

GPT-5.6 is proactive and persistent on multi-step tasks; define what each request authorizes. Compact policy:

> For requests to answer, explain, review, diagnose, or plan, inspect the relevant materials and report the result. Do not implement changes unless the request also asks for them. For requests to change, build, or fix, make the requested in-scope local changes and run relevant non-destructive validation without asking first. Require confirmation for external writes, destructive actions, purchases, or a material expansion of scope.

Name safe local actions explicitly (reading files, inspecting logs, editing in-scope code, running tests). State each rule once, in one place. Repeating "ask first," "do not mutate," or "wait for approval" causes unnecessary approval requests for safe actions. For long-running work, name the current layer (research, design, implementation, review, external coordination) so the model doesn't silently move between them.

## Tool routing

Expose only task-relevant tools. Descriptions should state what the tool does, when to use it, important return fields, and error behavior. When correctness depends on prerequisite retrieval: "Before taking an action, resolve required discovery, retrieval, and validation steps. Do not skip a prerequisite because the intended final state seems obvious." Parallelize independent reads; keep dependent work sequential; synthesize after parallel retrieval before acting. On empty/partial/suspiciously narrow tool results, try one or two meaningful fallbacks before concluding no result exists.

## Programmatic Tool Calling (PTC)

Best for bounded workflows where code processes several tool results or large intermediate outputs into a much smaller structured result. Multiple, parallel, or dependent calls alone do not justify it. Use for: filtering/joining/sorting/ranking/deduplication/aggregation; batching across similar records; repeated deterministic validation; large structured results reducible to a compact schema. Prefer direct calls when: one call suffices; intermediates are small; each result changes the next decision; an action needs approval; citations or native artifacts must be preserved; semantic judgment is needed between calls.

Don't use generic instructions like "use PTC efficiently." State the bounded stage, eligible tools, output schema, retry limit, stop condition, and handoff: "Use Programmatic Tool Calling only for the bounded record-reduction stage. Call only the documented read-only tools. Filter and deduplicate the intermediate results, then emit exactly the required compact schema with evidence fields. Retry transient failures at most twice. Use direct tool calls for approval, semantic judgment, citations, and final validation."

The `program_output` item and the final assistant `message` are separate outputs; test both, since a program can return correct records while the message omits a required field, citation, or caveat. Compare direct vs. programmatic on the same tasks; count lower resource use as a win only when responses still pass your evals.

## Grounding, citations, and retrieval budgets

Make citation behavior part of the prompt: what needs support, what counts as enough evidence, how to behave when evidence is missing. Absence of evidence should not automatically become a factual "no." Retrieval-budget sample: one broad search with short discriminative keywords; answer if the top results support the core request; retrieve again only when a required fact/owner/date/ID/source is missing, exhaustive coverage was requested, a specific artifact must be read, or an important claim would be unsupported; never retrieve again just to improve phrasing.

For research and synthesis: cite only retrieved sources; attach citations to the claims they support; label inference separately from supported facts; state conflicts between sources; narrow the answer or report missing evidence instead of guessing. For creative drafting: don't invent names, metrics, dates, roadmap status, customer outcomes, or product capabilities.

## Long-running workflows and state

Prompt for a short visible preamble before the first tool call, then sparse outcome-based updates at major phase changes. Don't ask the model to narrate routine tool calls: "Before tool calls for a multi-step task, send a one- or two-sentence user-visible update that states the first step. During the task, update only when a major phase begins or a finding changes the plan. Each update should state one concrete outcome and the next step."

Preserve assistant phase values when replaying history (automatic with `previous_response_id`; preserve them unchanged if replaying manually). Compact after major milestones, not every turn; keep the prompt functionally consistent after compaction; treat compacted items as opaque state. Persisted reasoning helps when the objective, assumptions, and priorities are stable across turns, but stale reasoning adds tokens and anchors the model to outdated approaches, so it is not an always-on optimization. Keep reusable prompt prefixes stable for caching; use explicit cache breakpoints only when measured behavior improves.

## Reasoning effort

Baseline first: preserve the current GPT-5.5/5.4 reasoning effort, test the same setting and one level lower on representative tasks. `low` for latency-sensitive work when quality holds; `medium` as the balanced start; `high`/`xhigh` only when evals show a meaningful gain; `max` reserved for the hardest quality-first workloads, never globally. Before raising effort, check whether the prompt is missing a success criterion, dependency rule, tool-routing rule, or verification loop.

## Frontend and visual tasks

Stronger layout and design judgment, but still provide product context, preserve the existing design system, and name the states and constraints that matter. Incremental changes: inspect and preserve existing design tokens/components/patterns; no extra features or decorative UI unless requested; preserve responsive behavior and expected states; render and inspect before finalizing. For vision/computer use/localization/OCR where spatial precision matters, choose image detail intentionally (original detail for large, dense, or coordinate-sensitive images when cost is justified).

## Check work before finishing

Give the model validating tools and state what validation matters. Coding: "After making changes, run the most relevant validation available: targeted tests for changed behavior; type checks or lint checks when applicable; build checks for affected packages; a minimal smoke test when full validation is too expensive. If validation cannot be run, explain why and describe the next best check." Visual artifacts: render, inspect layout/clipping/spacing/missing content, revise until it matches. Implementation plans should include requirements, named resources, state transitions or data flow, validation checks, failure behavior, privacy/security considerations, and open questions.

## Suggested prompt structure

Keep each section short; add detail only where it changes behavior:

> Role: [the model's function and context]
> Personality: [tone and collaboration style]
> Goal: [user-visible outcome]
> Success criteria: [what must be true before the final answer]
> Constraints: [policy, safety, business, evidence, and side-effect limits]
> Tools: [which tools to use, when, and what not to use]
> Output: [sections, length, format, and tone]
> Stop rules: [when to retry, fallback, abstain, ask, or stop]

## Prompt migration workflow

Moving an existing application to GPT-5.6: (1) switch the model, preserve current reasoning effort; (2) run representative evals before changing the prompt; (3) remove obsolete scaffolding, repeated instructions, irrelevant tools; (4) add only the smallest targeted instruction that fixes a measured regression; (5) re-run evals after each change. Don't rewrite a working prompt stack all at once, or you won't know whether a behavior change came from the model, effort setting, prompt, tools, or runtime. Debug regressions with a small set of real traces: identify the failure mode, find the instruction or contradiction that caused it, make a surgical edit, rerun the same cases.
