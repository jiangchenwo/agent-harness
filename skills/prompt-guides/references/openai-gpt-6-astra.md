# Using GPT-6 Astra (official guide, condensed)

- Source: https://developers.openai.com/api/docs/guides/latest-model
- Fetched: 2026-09-05. Append `.md` to the URL for the raw Markdown source.
- Covers: GPT-6 Astra model overview, new API capabilities, prompting guidance for behavior and writing style, and migration steps from earlier models.

## Introduction

GPT-6 Astra is OpenAI's most capable model, with top results on computer use, browsing, software engineering, science, and professional work. It handles multistep workflows across code, browsers, and professional software. Despite higher per-token pricing, it uses substantially fewer output tokens per task than earlier models, resulting in lower estimated API cost per task.

The model exercises care, respects task boundaries, and communicates transparently. It fills routine gaps using context, asks focused questions when the answer could change the outcome, incorporates new requirements, and tracks the broader task while handling side questions.

To use Astra, set `model` to `gpt-6-astra` in a Responses API request.

## What's new

- Async tool calling: The model can continue reasoning or call other tools while your application executes a tool. Set `async: true` on a function or custom tool and return its result using the original `call_id` when ready. See the Async tool calling guide for basic usage and a developer-defined wait-tool pattern.
- Mid-turn steering: Send additional user instructions while the model is working. Over a WebSocket connection, the Responses API preserves completed work and folds the update into a continuation. See the Mid-turn steering guide for the event flow and tool-result handling.
- Change reasoning mid-conversation while preserving cache: Add a `configuration_update` input item to raise or lower reasoning effort without rewriting the original prompt prefix. The updated effort applies until another `configuration_update` overrides it. See the reasoning guide for examples and compatibility limits.
- Misalignment monitoring: OpenAI systems asynchronously monitor for misalignment and trigger alerts when necessary. See the misalignment monitoring guide for details.
- Limitations: GPT-6 Astra does not support `none` reasoning effort. Fast mode is unavailable for GPT-6 Astra with EU data residency.

GPT-6 Astra also supports the capabilities from GPT-5.6, including computer use, Structured Outputs, streaming, Programmatic Tool Calling, multi-agent orchestration, prompt caching, persisted reasoning, compaction, and pro mode.

## Prompting best practices

GPT-6 Astra is more capable than GPT-5.6 Sol and shows behavior patterns that respond to prompting.

### GPT-6 Astra behavior

The model asks for clarification more often than earlier models (see initiative and follow-through). It follows longer instructions more reliably but reacts to unclear or conflicting guidance in skill files and `AGENTS.md` (instruction following). It defaults to detailed formatted output and may reuse phrases across sessions (personality and writing style). It delegates to subagents less often than multi-agent setups may require (subagent delegation). On small coding tasks it tends to run broader tests than the change warrants (testing and verification).

### Initiative and follow-through

The model stays coherent on long tasks but asks for clarification more often than earlier models. To encourage autonomous execution:

> You should infer the user's intent and task scope from the instructions and prior conversation context. Your job is to bias towards action and carry the user's intended task to completion.
>
> When the user expresses intent to perform new work or fix an existing issue, persist until the user's intended goal is complete. Progress autonomously towards the user's goal (e.g. creating isolated worktrees / checkouts if needed, resolving merge conflicts, read-only actions, creating draft PRs etc.) unless they are clearly destructive or irreversible.

To treat expressions of intent as instructions to act:

> When the user's prompt indicates a request for action, such as "can you...", "I want to...", "help me..." and similar expressions, treat these as instructions to do the work and take action. Do not stop at acknowledging capability (e.g. "Yes…"), proposing a plan, or offering to continue. Do not settle for a partial or "helpful enough" solution that does not fully satisfy the user's task to save time, effort or tokens. If a task requires sustained work, complete all the necessary work until the intended outcome is fulfilled.

To keep the model working until a concrete result is ready for review:

> Before asking the user clarifying questions, you should complete the work that is already authorized from context and necessary to make the proposed action concrete and reviewable. The user should be approving a concrete, reviewable result. For example, before deploying a change, writing to an external application, merging a PR or publishing a site, do all the required work first so that user approval is the final step. You don't need user permission for reversible tasks, read-only actions, reviews or fixes, or anything for which authorization is provided earlier in the session or strongly implied from the task instruction.
>
> Do not introduce unsolicited warnings, disclaimers, approval flows, or safety/compliance checklists due to hypothetical risk.

### Instruction following

The model follows longer instructions but reacts more to unclear or conflicting guidance in context. Make the priority of instructions explicit:

> The user's instructions take precedence over guidelines provided in a skill. If explicit user instructions conflict with a skill's instructions, prioritize the user's instructions.

To get transparency when a skill causes a pause or redirect:

> If a skill causes you to ask for permission or confirmation, pause, leave requested work unfinished, or diverge from the user's intent, name and link to the exact SKILL.md file you read, quote the relevant instruction, and briefly explain how it applies. Distinguish explicit skill requirements from your interpretation of guidelines.

### Personality and writing style

The model defaults to lists, tables, and Markdown. To get prose-first responses:

> Default to using clear, concise paragraphs, each developing one main idea. Use lists only when the information is genuinely parallel, sequential, or easier to compare, and avoid nested lists unless the hierarchy cannot be expressed clearly in prose. Use plain, simple language: familiar words, concrete examples, and precise verbs. Prefer active voice and direct statements.
>
> Make sure to state the main point clearly and early, then develop it with the explanation and detail the reader needs. Let each sentence build on what came before. Develop the points that matter and provide enough support to be useful.

For technical communication that stays readable:

> Use plain language over jargon, and reference technical details only to the degree that it helps illustrate an idea or your work to the user. Communicate complex concepts in a clear and cohesive manner, and calibrate your writing to the level of background knowledge assumed from the user's prompt and context.

To reduce filler phrases and stock language:

> Avoid using slop words or phrases like "Bottom Line:" in conclusions, "delve," "foster," "leverage," "it's worth noting," "importantly," "Question? Answer." or "This isn't about X. It's about Y.", "genuinely" or hyphenated compound descriptions and adjectives. Do not use concluding summary statements such as "In short:..", "The simplest mental model is:...".
>
> State the intended action directly. Avoid adding what you won't do, what will remain unchanged, or how you'll separate or categorize results. Do not use contrastive framing such as "X, not Y" or "X—not Y" that introduces an unprompted alternative that the user didn't ask about. Avoid invented compound labels like "exact-head checks" and "editorial-row layouts", vague qualifiers, and canned transitions; use plain verbs and prepositions to state the actual relationship directly.

### Subagent delegation

To tune how much the model delegates in a multi-agent setup:

> If at any point you can parallelize work by delegating tasks to another agent (no matter if you are the root or subagent), you should do so using collaboration tools if it could save time or improve quality.

To improve legibility of inter-agent messages:

> Messages that you send to other agents and your final answer may be read by a human, so ensure they are legible. Always put proper spaces between words and/or numbers.

### Testing and verification

To calibrate testing scope on coding tasks:

> Do not write tests for reversible, low-impact changes that mirror the implementation. If you do choose to verify your work with tests, make sure that the tests are meaningful and necessary to verify implementation.
>
> Run tests appropriate to the change and complete required checks. Once those pass, broaden or repeat testing only when new changes, failures, or unresolved concerns justify it; otherwise, continue toward completing the task.

## Migration quickstart

### Migrate with Codex

The OpenAI Docs skill (available at the OpenAI skills repository) can apply the recommended changes automatically. Invoke it in Codex with:

> $openai-docs migrate this project to GPT-6 Astra

### Update API and model parameters

Set `model` to `gpt-6-astra`, then check the following:

- Reasoning effort: If you currently use `none` or `minimal`, start with `low` and compare results. Otherwise, preserve your current reasoning effort. Use `reasoning.effort` in Responses or `reasoning_effort` in Chat Completions. `none` is not supported.
- Tool calling: Use the Responses API. GPT-6 Astra supports Chat Completions, but tool calling requires Responses.
- Unsupported parameters: Remove `temperature`, `top_p`, and `top_logprobs`. For Chat Completions, also remove `logprobs`. For Responses, remove `message.output_text.logprobs` from `include`.
- Fast mode: For EU data residency, use Standard processing. `service_tier: "fast"` and `service_tier: "priority"` are not supported with EU data residency. Fast mode for GPT-6 Astra does not include a latency SLA.
- Changing reasoning effort: Use `configuration_update` input items in standard, single-agent requests to change effort between responses. Keep request-level `reasoning.effort` unchanged to preserve the prompt prefix for caching. Check the compatibility limits before adopting this.
- Prompt caching: When migrating from GPT-5.5 or earlier, replace `prompt_cache_retention` with `prompt_cache_options.ttl` set to `"30m"`. Review the prompt caching changes, including cache boundaries and cache-write billing.
- Unnecessary approval pauses: Use the initiative and follow-through prompts above to encourage more autonomous execution. See Prompting best practices for guidance on instruction following, writing style, subagent delegation, and testing.
