# Claude prompting best practices (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Fetched: 2026-11-05. Append `.md` to the URL for the raw Markdown source; fetch it when you need full SDK code samples or exact XML example blocks, which this condensation omits.
- Covers: techniques that apply across all current Claude models (Claude Fable 5.1, Mythos 5.1, Fable 5, Mythos 5, Opus 5.5, Opus 5, Opus 4.8/4.7/4.6, Sonnet 5.5, Sonnet 5, Sonnet 4.6, Haiku 5.5, Haiku 4.5). The page points to separate per-model guides for model-specific behavior.

## Contents

1. Model-specific guidance
2. General principles
3. Output and formatting
4. Tool use
5. Thinking and reasoning
6. Agentic systems
7. Capability-specific tips
8. Migration considerations

## Model-specific guidance

Each of these models has its own prompting page. The page says to read the one for your model first, then the techniques that follow. Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.

| Model | What the page says is different |
|---|---|
| Claude Fable 5.1 and Mythos 5.1 | From Fable 5: effort levels, finishing long tasks, user-facing progress updates, passing thinking blocks back unchanged, tool-call batching in agent loops, search triggering at low effort, formatting, and writing density. |
| Claude Fable 5 and Mythos 5 | From Opus 4.8: effort levels, instruction following, long-run progress claims, memory systems, and the `reasoning_extraction` refusal category. |
| Claude Sonnet 5.5 | From Sonnet 5: effort calibration, initiative and scope, running without up-front thinking, JSON output on reasoning tasks, user-facing progress updates, tool use in chat, mid-turn user messages, verification on coding tasks, tool-call handling, complex visual inputs, and safeguard refusals. |
| Claude Sonnet 5 | From Sonnet 4.6: response length, effort and thinking-depth calibration, tool use triggering, literal instruction following, and design and frontend defaults. |
| Claude Opus 5.5 | From Opus 5: effort calibration, prompts written for thinking disabled, user-facing progress updates, safeguard refusals, and tools for complex visual inputs. |
| Claude Opus 5 | From prior Opus models: response length and verbosity, user-facing progress updates, written deliverable length, task scope and over-verification, subagent control, and self-correction. |
| Claude Opus 4.8 | Response length, effort and thinking-depth calibration, tool use triggering, literal instruction following, subagent control, and design and frontend defaults. |
| Claude Haiku 5.5 | From Haiku 4.5: effort levels, search, JSON output with your own tools, early stopping in long agent prompts, verification on coding tasks, mid-turn user messages, system prompt adherence in chatbots, and safeguard refusals. |

## General principles

**Be clear and direct.** Claude responds well to clear, explicit instructions. If you want "above and beyond" behavior, explicitly request it rather than relying on the model to infer it. Think of Claude as a brilliant but new employee who lacks context on your norms and workflows. Golden rule: show your prompt to a colleague with minimal context and ask them to follow it. If they'd be confused, Claude will be too. Be specific about output format and constraints; give sequential steps as numbered lists when order or completeness matters. Example: "Create an analytics dashboard" is less effective than "Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation."

**Add context to improve performance.** Explaining why a behavior matters helps Claude generalize. "NEVER use ellipses" is less effective than "Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them."

**Use examples effectively.** A few well-crafted examples (few-shot/multishot) improve accuracy and consistency. Make them relevant (mirror the actual use case), diverse (cover edge cases, vary enough to avoid unintended patterns), and structured (wrap in `<example>` tags, multiple in `<examples>`). Include 3 to 5 examples for best results.

**Structure prompts with XML tags.** Tags like `<instructions>`, `<context>`, `<input>` help Claude parse prompts that mix instructions, context, examples, and variable inputs. Use consistent, descriptive tag names; nest tags when content has a natural hierarchy.

**Give Claude a role.** Even a single system-prompt sentence ("You are a helpful coding assistant specializing in Python.") focuses behavior and tone.

**Long context prompting** (20k+ token inputs):

- Put longform data at the top, above the query, instructions, and examples. Queries at the end can improve response quality by up to 30 percent in tests, especially with complex multidocument inputs.
- Wrap each document in `<document>` tags with `<document_content>` and `<source>` subtags.
- Ground responses in quotes: ask Claude to quote relevant parts of the documents first (e.g. in `<quotes>` tags) before carrying out its task.

**Model self-knowledge.** To have Claude identify itself correctly, state it: "The assistant is Claude, created by Anthropic. The current model is Claude Opus 5.5." For apps that must emit model strings: "When an LLM is needed, please default to Claude Opus 5.5 unless the user requests otherwise. The exact model string for Claude Opus 5.5 is claude-opus-5-5."

## Output and formatting

**Communication style.** Current models are more direct and grounded, more conversational, and less verbose than previous generations; they may skip summaries after tool calls. For more visibility: "After completing a task that involves tool use, provide a quick summary of the work you've done." Exception: Claude Opus 5's user-facing responses run longer by default, and effort does not reliably change visible response length. Prompt explicitly for conciseness; the page links to the Opus 5 prompting guide for the details. Claude Fable 5.1 has the opposite tendency during agentic work: it writes fewer user-facing updates between tool calls. Ask for progress text explicitly, and remove any instruction telling it to keep that text brief (the Fable 5.1 guide's "Ask for user-facing progress updates" section).

**Control response format.** Four techniques, in order of usefulness:

1. Tell Claude what to do instead of what not to do ("Your response should be composed of smoothly flowing prose paragraphs," not "Do not use markdown").
2. Use XML format indicators ("Write the prose sections in `<smoothly_flowing_prose_paragraphs>` tags").
3. Match your prompt style to the desired output: removing markdown from your prompt reduces markdown in the output.
4. For fine control, give detailed guidance. The official sample wraps it in `<avoid_excessive_markdown_and_bullet_points>` tags: write long-form content in flowing prose with complete paragraphs; reserve markdown for inline code, code blocks, and simple headings; avoid bold/italics; no ordered or unordered lists unless the items are truly discrete or the user asked; never output a series of overly short bullet points. Claude Fable 5.1 already formats less than earlier models, so on that model a block like this can suppress structure the content needs; remove it, or replace it with the shorter rule in the Fable 5.1 guide's "Formatting in chat" section.

**LaTeX.** Current models default to LaTeX for math. For plain text: "Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents)."

**Document creation.** Strong instruction following for presentations, animations, and visual documents; request design elements explicitly ("Include thoughtful design elements, visual hierarchy, and engaging animations where appropriate.").

**Prefilled responses are gone.** Starting with Claude 4.6 models and Claude Mythos Preview, a prefilled assistant message on the last turn returns a 400 error. Migrations:

- Output formatting → Structured Outputs, or ask the model to conform to the schema (with retries); classification → tools with an enum field or structured outputs.
- Eliminating preambles → "Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...'", or XML tags / structured outputs / tool calling; strip stragglers in post-processing.
- Steering around bad refusals → no longer needed; clear prompting in the user message suffices.
- Continuations → move to the user message: "Your previous response was interrupted and ended with `[previous_response]`. Continue from where you left off."
- Context hydration → inject reminders into the user turn, hydrate through tools, or during context compaction.

## Tool use

**Be explicit for action.** Current models follow instructions precisely: "Can you suggest some changes?" gets suggestions, not edits. Say "Change this function to improve its performance." To shift the default toward acting, the official `<default_to_action>` sample: "By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly." The inverse `<do_not_act_before_instructions>` sample: "Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them."

**Dial back aggressive triggering language.** Claude Opus 4.5 and 4.6 are more responsive to the system prompt; prompts written to fix undertriggering ("CRITICAL: You MUST use this tool when...") now overtrigger. Use normal register: "Use this tool when...".

**Parallel tool calls.** Current models run independent tool calls in parallel by default (speculative searches, multi-file reads, parallel bash). To push toward ~100%, the official `<use_parallel_tool_calls>` sample: "If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls." To reduce it: "Execute operations sequentially with brief pauses between each step to ensure stability." On Claude Fable 5.1 in long agent loops, send the parallel-calls instruction as a turn-scoped system message after each round of tool results (the Fable 5.1 guide's "Batch independent tool calls in agent loops" section).

## Thinking and reasoning

**Overthinking and excessive thoroughness.** Claude Opus 4.6 does more upfront exploration, especially at higher `effort`. If your prompts encouraged thoroughness, tune them down: replace blanket defaults ("Default to using [tool]") with targeted instructions ("Use [tool] when it would enhance your understanding"); remove over-prompting ("If in doubt, use [tool]" now causes overtriggering); use a lower `effort` as fallback. To constrain rumination: "When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning."

**Thinking defaults by model** (the detail most likely to bite in cross-model prompts):

- Opus 4.6 through Opus 4.8 and Sonnet 4.6: thinking is **off** when the `thinking` parameter is omitted; enable adaptive thinking with `thinking: {type: "adaptive"}`.
- Opus 5, Sonnet 5.5, Sonnet 5, and Haiku 5.5: thinking is **on** by default, so a response can begin with `thinking` blocks. Select content blocks by `type` rather than by position, and leave room in `max_tokens` for thinking.
- Opus 5 and Haiku 5.5: thinking can be disabled only at effort `high` or lower.
- Sonnet 5.5: the lowest thinking setting is `between_tools`, accepted at effort `high` or lower.
- Fable 5.1, Mythos 5.1, Fable 5, Mythos 5, and Opus 5.5: thinking is **always on**, regardless of the `thinking` parameter.

**Adaptive thinking.** Claude 4.6+ and Mythos Preview use adaptive thinking: the model decides when and how much to think, calibrated by the `effort` parameter and query complexity. On Fable 5.1, Mythos 5.1, Fable 5, Mythos 5, and Opus 5.5 it is the only mode. In internal evaluations it reliably beats extended thinking. Use it for agentic workloads: multistep tool use, complex coding, long-horizon loops. Older models use manual extended thinking with `budget_tokens`; the per-model configuration table at /docs/en/build-with-claude/thinking-troubleshooting#supported-models lists which configuration each model accepts.

**budget_tokens is deprecated.** Still functional on Opus 4.6 / Sonnet 4.6; returns a 400 error on Claude 4.7 and later. Control depth with `effort` (in `output_config`), cap cost with `max_tokens`.

**Guiding reasoning:**

- Prefer general instructions over prescriptive steps: "think thoroughly" often beats a hand-written plan; Claude's reasoning frequently exceeds what a human would prescribe.
- Multishot examples work with thinking: worked examples shape how Claude approaches similar problems in its own thinking blocks. Present each example as a problem, the method to apply, and the expected answer.
- Manual chain-of-thought is a fallback when thinking is off: ask Claude to think through the problem before it answers and put the final answer in `<answer>` tags. On Fable 5.1, Fable 5, Opus 5.5, Opus 5, and Sonnet 5.5, rely on thinking instead, at lower effort if cost or latency matters: a prompt that asks the model to write out its reasoning, for example in `<thinking>` tags, may be declined (refusals-and-fallback page, "Keep reasoning in thinking blocks").
- Ask Claude to self-check: "Before you finish, verify your answer against [test criteria]." Reliable for coding and math, except on Claude Opus 5, which verifies well unprompted; carried-over verification instructions cause over-verification there (remove them, don't rewrite them).
- With thinking disabled, Claude Opus 4.5 is sensitive to the word "think"; use "consider," "evaluate," "reason through."
- To damp adaptive-thinking triggering: "Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multistep reasoning. When in doubt, respond directly."

## Agentic systems

**Long-horizon work and state tracking.** Current models maintain orientation across extended sessions by making steady incremental progress and can span multiple context windows by saving state and continuing fresh.

**Context awareness.** Sonnet 5, Sonnet 4.6, Sonnet 4.5, and Haiku 4.5 track their remaining context window. If your harness compacts context or persists state to files, say so, or the model may wrap up early. Official sample: "Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining." The memory tool pairs well with context awareness for managing context transitions.

**Multi-context-window workflows:** use a different prompt for the first window (set up framework, tests, setup scripts); have the model write tests in a structured format (`tests.json`) and remind it: "It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality."; encourage setup scripts (`init.sh`); consider a fresh window over compaction (the models discover state from the filesystem well; be prescriptive: "Call pwd; you can only read and write files in this directory.", "Review progress.txt, tests.json, and the git logs.", "Manually run through a fundamental integration test before moving on to implementing new features."); provide verification tools (the computer use tool, the browser use tool, or a browser automation MCP server); encourage full use of context: "This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task."

**State management:** structured formats (JSON) for state data, freeform text for progress notes, git for checkpoints and history, and explicit emphasis on incremental progress.

**Balancing autonomy and safety.** Without guidance, Claude Opus 4.6 may take hard-to-reverse actions (deleting files, force-pushing, posting externally). The official sample asks the model to weigh reversibility: take local, reversible actions freely; confirm before destructive operations (deleting files/branches, dropping tables, rm -rf), hard-to-reverse ones (git push --force, git reset --hard), or actions visible to others (pushing, PR comments, messages, shared infrastructure); and never use destructive actions as a shortcut around obstacles (no --no-verify, no discarding unfamiliar files).

**Research.** Provide clear success criteria; encourage source verification across multiple sources. For complex research: "Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically."

**Subagent orchestration.** Current models delegate to subagents proactively when subagent tools are well defined, with no explicit instruction needed. Watch for overuse: Opus 4.6 has a strong predilection for subagents (spawning them where a direct grep is faster), and Opus 5 also delegates more readily than prior models. Damping sample: "Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating."

**Chain complex prompts.** Adaptive thinking plus subagent orchestration handle most multistep reasoning internally; explicit chaining (sequential API calls) remains useful to inspect intermediate outputs or enforce a pipeline. The most common pattern is self-correction: draft → review against criteria → refine, each a separate call you can log, evaluate, or branch.

**Reduce file creation.** The models use temporary files (especially Python scripts) as scratchpads, which improves agentic coding outcomes. To keep trees clean: "If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task."

**Overeagerness.** Opus 4.5 and 4.6 tend to overengineer. The official minimal-solutions sample opens "Avoid over-engineering. Only make changes that are directly requested or clearly necessary," then sets four scopes: don't add features/refactors beyond the ask; don't document code you didn't change; don't add defensive handling for impossible scenarios (validate only at system boundaries); don't create abstractions for one-time operations or hypothetical futures. "The right amount of complexity is the minimum needed for the current task."

**Tests and hardcoding.** Claude can overfocus on making tests pass. Countersample: "Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally. Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles. If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable."

**Minimizing hallucinations.** The official `<investigate_before_answering>` sample: "Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers."

## Capability-specific tips

**Vision.** Opus 4.5 and 4.6 improved image processing, data extraction, and multi-image contexts; this carries into computer use. The models can also analyze video broken into frames. Giving Claude a crop tool or agent skill (letting it "zoom" into image regions) gives consistent uplift; Anthropic publishes a crop-tool recipe in the platform cookbook.

**Frontend design.** Without guidance the models converge on generic "AI slop" aesthetics. The official `<frontend_aesthetics>` sample pushes distinctive typography (no Arial/Inter/Roboto), committed color themes with CSS variables, high-impact motion (one well-orchestrated staggered page load over scattered micro-interactions), layered backgrounds over solid colors, and warns the model it still converges on common choices (e.g. Space Grotesk) across generations. Full text at the source URL, or the frontend-design skill in the claude-code repo. For frontend design work outside the API, the page points to Claude Design (support.claude.com/en/articles/14604416-get-started-with-claude-design).

**Medication dosing.** If people might ask your application about medicine doses, add the block for the model your application calls to your system prompt, exactly as written. The block has Claude give a child's dose of an over-the-counter medicine from the product's label, not from memory. With web search, Claude checks the label before answering, which adds latency and search costs. Without web search, Claude points the person to the label's dosing chart, a pharmacist, or the child's doctor instead of giving a dose.

For Claude Fable 5.1, Fable 5, Opus 5.5, Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 5.5, and Sonnet 5:

> <medical_guidance>
> This applies only when the person explicitly says the medicine is for a child, and isn't a healthcare professional asking for work.
> Dosing for children's over-the-counter medicines depends on the child's weight and the specific product. For infants and toddlers, a small error can cause real harm. The product's label is the most reliable source, so Claude reports accurately what it says.
> If neither the child's age or weight is given, Claude asks for both. If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
> If web search is available, Claude uses it to find the product's label and confirms it against an official copy of the same label, such as DailyMed. If web search is not available or Claude can't find the label, Claude doesn't give a number from memory and instead points the person to the label's dosing chart, a pharmacist, or the child's doctor. Claude shares what the label states for the child's age and weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
> </medical_guidance>

For Claude Haiku 5.5:

> <medical_guidance>
> This applies only when the person explicitly says the medicine is for a child, and isn't a healthcare professional asking for work.
> Dosing for children's over-the-counter medicines depends on the child's age or weight and the specific product. For infants and toddlers, a small error can cause real harm. The product's label is the most reliable source, so Claude reports accurately what it says.
> If neither the child's age or weight is given, Claude asks for both. If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
> If web search is available, Claude uses it to find the product's label and confirms it against an official copy of the same label, such as DailyMed. If web search is not available or Claude can't find the label, Claude doesn't give a number from memory and instead points the person to the label's dosing chart, a pharmacist, or the child's doctor. Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
> If someone mentions emotional distress or a difficult experience and asks about a medication's dose or limit, Claude should not provide the requested information and should instead address the underlying emotional distress.
> </medical_guidance>

For Claude Sonnet 4.6:

> <medical_guidance>
> If web search is available, Claude uses it to find the children's medicine label and confirms it against an official copy of the same label, such as DailyMed. Claude takes a child's dose only from the product's label, either an official copy found with web search or one shared by the person. Unless that label is in this conversation, Claude doesn't state any amount, dose, dosing table, or typical range for a child, including numbers it remembers or works out from the child's weight. Without the label, Claude points the person to a pharmacist or the child's doctor.
> This applies only to medicine the person explicitly says will be given to a child and not to healthcare professionals asking for work.
> If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
> If neither the child's age nor weight is given, Claude asks for both.
> Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
> </medical_guidance>

For Claude Haiku 4.5:

> <medical_guidance>
> When someone asks about a medicine for a child and web search is available, Claude uses it to find that medicine's label and confirms it against an official copy, such as DailyMed.
> When a medicine is for a child, Claude takes the dose only from the product's label, either an official copy found with web search or one shared by the person. Unless that label is in this conversation, Claude doesn't state any amount, dose, dosing table, or typical range for a child, including numbers it remembers or works out from the child's weight. Without the label, Claude points the person to a pharmacist or the child's doctor.
> Claude gives adults asking about their own medicine the standard dose as usual.
> If no child is mentioned, Claude answers as it would for an adult.
> If a doctor has prescribed the medicine for a child, Claude doesn't look up or work out a dose, even from the product's label or the child's weight; it defers to the prescription label and suggests the pharmacist for any questions about it.
> If neither the child's age nor weight is given, Claude asks for both.
> Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the child's age or weight is above or below what the label's chart includes, or the label says not to use the product, Claude says so and refers the person to their doctor or pharmacist. Claude doesn't give doses from charts other than the product's label.
> </medical_guidance>

## Migration considerations

Moving prompts from earlier generations to current models:

1. Be specific about desired behavior and output.
2. Add quality modifiers ("Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.").
3. Request animations and interactive elements explicitly.
4. Update thinking configuration to adaptive thinking + `effort` (no `budget_tokens`).
5. Migrate away from prefilled responses (see Output and formatting above).
6. Tune anti-laziness prompting down: 4.6+ models are proactive and overtrigger on guidance written for older, lazier models.
7. Pass thinking blocks back unchanged and keep history append-only: append each assistant turn exactly as the API returned it, thinking blocks included. On Claude 5.1 and later models, editing earlier messages, rebuilding `system` or `tools`, or summarizing older turns in place between requests invalidates every later thinking block, so move those changes to mid-conversation system messages and server-side context management (the Fable 5.1 guide's "Keep the conversation history append-only" section).

For detailed migration steps, see the migration guide at /docs/en/about-claude/models/migration-guide.

**Migrating to Claude Sonnet 5.5.** The Sonnet 5.5 migration guide (https://platform.claude.com/docs/en/models/sonnet-5-5/migration-guide) covers the breaking changes for code written for Claude Sonnet 5 and the steps from Claude Sonnet 4.6 and earlier, including the removal of manual extended thinking (`budget_tokens`).
