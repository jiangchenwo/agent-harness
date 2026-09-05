# OpenAI prompt engineering (official guide, condensed)

- Source: https://developers.openai.com/api/docs/guides/prompt-engineering
- Fetched: 2026-08-12. Append `.md` to the URL for the raw Markdown source; fetch it for SDK code samples and full prompt examples this condensation trims.
- Covers: all OpenAI API models (GPT and reasoning families). The page points to separate model-specific guidance for the latest model.

## Choosing a model

- **Reasoning models** generate an internal chain of thought; they excel at complex tasks and multi-step planning, and are generally slower and more expensive.
- **GPT models** are fast, cost-efficient, and highly intelligent, but benefit from more explicit instructions about how to accomplish tasks.
- **Large vs. mini/nano models** trade intelligence for speed and cost.

When in doubt, `gpt-5.6` is the recommended general-purpose default. Because behavior varies by model type and even snapshot: pin production applications to specific model snapshots, and build tests and evaluation suites so you can monitor prompt behavior as you iterate or upgrade.

## Message roles and instruction following

Instructions can be given with differing levels of authority via the `instructions` API parameter or message roles. The `instructions` parameter sets high-level behavior (tone, goals, examples of correct responses) and takes priority over the `input` prompt. Note: `instructions` applies only to the current request; with `previous_response_id`, prior turns' instructions are not in context.

Role priority per the OpenAI model spec: `developer` messages (from the application developer) are prioritized ahead of `user` messages, which are prioritized ahead of model-generated `assistant` messages. Mental model: `developer` messages are the function definition; `user` messages are the arguments passed to it.

## Version prompts in code

Store production prompts in application code, not reusable prompt objects (the `v1/prompts` API is deprecated: de-emphasized June 3, 2026, shutdown scheduled November 30, 2026). Keep prompt builders in a small module near the feature; use typed arguments or schemas for dynamic values; pass generated `instructions`/`input` directly to the Responses API; add fixtures, tests, and eval checks before changing production prompts; roll out via your deployment system with feature flags for staged releases.

## Message formatting with Markdown and XML

Markdown headers and lists mark sections and hierarchy; XML tags delineate where a piece of content begins and ends, with attributes for metadata your instructions can reference. A developer message usually contains, in order:

1. **Identity:** purpose, communication style, high-level goals.
2. **Instructions:** rules to follow, what to do and never do; subsections as needed (e.g. how to call functions).
3. **Examples:** possible inputs with desired outputs.
4. **Context:** proprietary or task-specific data. Best positioned near the end, since it varies between requests.

**Prompt caching:** keep content reused across requests at the beginning of the prompt and among the first API parameters in the request body to maximize cost and latency savings.

## Few-shot learning

Steer the model with a handful of input/output examples in the prompt instead of fine-tuning; the model picks up the pattern implicitly. Show a diverse range of possible inputs with desired outputs. Typically placed in the developer message. The official example pairs `<product_review id="example-N">` inputs with `<assistant_response id="example-N">` outputs for a sentiment classifier constrained to one-word answers.

## Include relevant context (RAG)

Add context to give the model proprietary data or constrain it to specific resources, via vector-database retrieval pasted into the prompt or the built-in file search tool. Plan for the context window (low-100k range up to 1M tokens depending on model).

## Prompting current GPT-5 series models

GPT models like `gpt-5.6` benefit from precise instructions that explicitly provide the logic and data required for the task. The page links to the GPT-5.6 prompting guide for the full treatment; practical reminders:

**Coding.** Define the agent's role and workflow; enforce structured tool use with concrete invocation examples; require testing (unit tests, Python commands) and careful patch validation, since tools like `apply_patch` may return "Done" even on failure; set Markdown standards (inline code, fences, lists, tables; backticks for file paths, functions, classes).

**Front-end engineering.** Recommended libraries: Tailwind CSS, shadcn/ui, Radix Themes (styling); Lucide, Material Symbols, Heroicons (icons); Motion (animation). For zero-to-one web apps, the official sample prompt has the model build a hidden rubric of 5 to 7 categories, iterate against it until every category meets the highest standard, and avoid external dependencies like Next.js or React. For large codebases, add instruction categories: principles (visual quality, modular components), UI/UX (typography, colors, spacing, interaction states, accessibility), structure (file/folder layout), components (reusable wrappers, backend-call separation), pages (layout templates), and agent instructions (confirm assumptions, scaffold, enforce standards, test states, document).

**Agentic tasks.** Three core practices: plan thoroughly for complete resolution, add preambles for major tool decisions, and track progress with a TODO tool. Official persistence sample:

> Remember, you are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user. Decompose the user's query into all required sub-requests, and confirm that each is completed. Do not stop after completing only part of the request. Only terminate your turn when you are sure that the problem is solved... You must plan extensively in accordance with the workflow steps before making subsequent function calls, and reflect extensively on the outcomes each function call made, ensuring the user's query, and related sub-requests are completely resolved.

Preamble sample: "Before you call a tool explain why you are calling it", but only at notable steps.

## Prompting reasoning models

Reasoning models do better with high-level guidance; GPT models benefit from very precise instructions. Analogy from the guide: a reasoning model is a senior co-worker, someone you give a goal and trust to work out the details; a GPT model is a junior co-worker who performs best with explicit instructions for a specific output. A dedicated reasoning-best-practices guide exists at /api/docs/guides/reasoning-best-practices.

## Other notes

- Model text output is not guaranteed at `output[0].content[0].text`: the `output` array can contain tool calls and reasoning items. SDKs provide `output_text` as a convenience aggregate.
- For schema-conforming JSON, use Structured Outputs.
