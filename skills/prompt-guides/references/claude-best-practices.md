# Claude prompting best practices (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Fetched: 2026-08-12. Append `.md` to the URL for the raw Markdown source; fetch it when you need full SDK code samples or exact XML example blocks, which this condensation omits.
- Covers: all current Claude models (Claude Fable 5, Mythos 5, Opus 5, Opus 4.8/4.7/4.6, Sonnet 5, Sonnet 4.6, Haiku 4.5). Model-specific deltas live in the sibling `claude-*.md` files.

## Contents

1. General principles
2. Output and formatting
3. Tool use
4. Thinking and reasoning
5. Agentic systems
6. Capability-specific tips
7. Migration considerations

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

**Model self-knowledge.** To have Claude identify itself correctly, state it: "The assistant is Claude, created by Anthropic. The current model is Claude Opus 5." For apps that must emit model strings: "When an LLM is needed, please default to Claude Opus 5 unless the user requests otherwise. The exact model string for Claude Opus 5 is claude-opus-5."

## Output and formatting

**Communication style.** Current models are more direct and grounded, more conversational, and less verbose than previous generations; they may skip summaries after tool calls. For more visibility: "After completing a task that involves tool use, provide a quick summary of the work you've done." Exception: Claude Opus 5's user-facing responses run longer by default, and effort does not reliably change visible response length. Prompt explicitly for conciseness (see `claude-opus-5.md`).

**Control response format.** Four techniques, in order of usefulness:

1. Tell Claude what to do instead of what not to do ("Your response should be composed of smoothly flowing prose paragraphs," not "Do not use markdown").
2. Use XML format indicators ("Write the prose sections in `<smoothly_flowing_prose_paragraphs>` tags").
3. Match your prompt style to the desired output: removing markdown from your prompt reduces markdown in the output.
4. For fine control, give detailed guidance. The official sample wraps it in `<avoid_excessive_markdown_and_bullet_points>` tags: write long-form content in flowing prose with complete paragraphs; reserve markdown for inline code, code blocks, and simple headings; avoid bold/italics; no ordered or unordered lists unless the items are truly discrete or the user asked; never output a series of overly short bullet points.

**LaTeX.** Current models default to LaTeX for math. For plain text: "Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters."

**Document creation.** Strong instruction following for presentations, animations, and visual documents; request design elements explicitly ("Include thoughtful design elements, visual hierarchy, and engaging animations where appropriate.").

**Prefilled responses are gone.** Starting with Claude 4.6 models and Claude Mythos Preview, a prefilled assistant message on the last turn returns a 400 error. Migrations:

- Output formatting → Structured Outputs, or ask the model to conform to the schema (with retries); classification → tools with an enum field or structured outputs.
- Eliminating preambles → "Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...'", or XML tags / structured outputs / tool calling; strip stragglers in post-processing.
- Steering around bad refusals → no longer needed; clear prompting in the user message suffices.
- Continuations → move to the user message: "Your previous response was interrupted and ended with `[previous_response]`. Continue from where you left off."
- Context hydration → inject reminders into the user turn, hydrate through tools, or during context compaction.

## Tool use

**Be explicit for action.** Current models follow instructions precisely: "Can you suggest some changes?" gets suggestions, not edits. Say "Change this function to improve its performance." To shift the default toward acting, the official `<default_to_action>` sample: "By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing." The inverse `<do_not_act_before_instructions>` sample: "Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action."

**Dial back aggressive triggering language.** Claude Opus 4.5 and 4.6 are more responsive to the system prompt; prompts written to fix undertriggering ("CRITICAL: You MUST use this tool when...") now overtrigger. Use normal register: "Use this tool when...".

**Parallel tool calls.** Current models run independent tool calls in parallel by default (speculative searches, multi-file reads, parallel bash). To push toward ~100%, the official `<use_parallel_tool_calls>` sample: "If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel... However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls." To reduce it: "Execute operations sequentially with brief pauses between each step to ensure stability."

## Thinking and reasoning

**Overthinking and excessive thoroughness.** Claude Opus 4.6 does more upfront exploration, especially at higher `effort`. If your prompts encouraged thoroughness, tune them down: replace blanket defaults ("Default to using [tool]") with targeted instructions ("Use [tool] when it would enhance your understanding"); remove over-prompting ("If in doubt, use [tool]" now causes overtriggering); use a lower `effort` as fallback. To constrain rumination: "When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning."

**Thinking defaults by model** (the detail most likely to bite in cross-model prompts):

- Opus 4.6 through Opus 4.8 and Sonnet 4.6: thinking is **off** when the `thinking` parameter is omitted; enable adaptive thinking with `thinking: {type: "adaptive"}`.
- Opus 5 and Sonnet 5: thinking is **on** by default; on Opus 5 it can be disabled only at effort `high` or lower.
- Fable 5 and Mythos 5: thinking is **always on**, regardless of the `thinking` parameter.

**Adaptive thinking.** Claude 4.6+ and Mythos Preview use adaptive thinking: the model decides when and how much to think, calibrated by the `effort` parameter and query complexity. In internal evaluations it reliably beats extended thinking. Use it for agentic workloads: multistep tool use, complex coding, long-horizon loops.

**budget_tokens is deprecated.** Still functional on Opus 4.6 / Sonnet 4.6; returns a 400 error on Claude 4.7 and later. Control depth with `effort` (in `output_config`), cap cost with `max_tokens`.

**Guiding reasoning:**

- Prefer general instructions over prescriptive steps: "think thoroughly" often beats a hand-written plan; Claude's reasoning frequently exceeds what a human would prescribe.
- Multishot examples work with thinking: `<thinking>` tags inside few-shot examples teach the reasoning pattern.
- Manual chain-of-thought is a fallback when thinking is off (structure with `<thinking>` and `<answer>` tags). On Opus 5, prefer keeping thinking enabled at lower effort instead; with thinking disabled it can leak internal XML tags into visible output.
- Ask Claude to self-check: "Before you finish, verify your answer against [test criteria]." Reliable for coding and math, except on Claude Opus 5, which verifies well unprompted; carried-over verification instructions cause over-verification there (remove them, don't rewrite them).
- With thinking disabled, Claude Opus 4.5 is sensitive to the word "think"; use "consider," "evaluate," "reason through."
- To damp adaptive-thinking triggering: "Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multistep reasoning. When in doubt, respond directly."

## Agentic systems

**Long-horizon work and state tracking.** Current models maintain orientation across extended sessions by making steady incremental progress and can span multiple context windows by saving state and continuing fresh.

**Context awareness.** Sonnet 5, Sonnet 4.6, Sonnet 4.5, and Haiku 4.5 track their remaining context window. If your harness compacts context or persists state to files, say so, or the model may wrap up early. Official sample: "Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns... Never artificially stop any task early regardless of the context remaining."

**Multi-context-window workflows:** use a different prompt for the first window (set up framework, tests, setup scripts); have the model write tests in a structured format (`tests.json`) and tell it not to remove or edit them; encourage setup scripts (`init.sh`); consider a fresh window over compaction (the models discover state from the filesystem well; be prescriptive: "Review progress.txt, tests.json, and the git logs"); provide verification tools (Playwright MCP, computer use); encourage full use of context ("It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work").

**State management:** structured formats (JSON) for state data, freeform text for progress notes, git for checkpoints and history, and explicit emphasis on incremental progress.

**Balancing autonomy and safety.** Without guidance, Claude Opus 4.6 may take hard-to-reverse actions (deleting files, force-pushing, posting externally). The official sample asks the model to weigh reversibility: take local, reversible actions freely; confirm before destructive operations (deleting files/branches, dropping tables, rm -rf), hard-to-reverse ones (git push --force, git reset --hard), or actions visible to others (pushing, PR comments, messages, shared infrastructure); and never use destructive actions as a shortcut around obstacles (no --no-verify, no discarding unfamiliar files).

**Research.** Provide clear success criteria; encourage source verification across multiple sources. For complex research: "As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency."

**Subagent orchestration.** Current models delegate to subagents proactively when subagent tools are well defined, with no explicit instruction needed. Watch for overuse: Opus 4.6 has a strong predilection for subagents (spawning them where a direct grep is faster), and Opus 5 also delegates more readily than prior models. Damping sample: "Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating."

**Chain complex prompts.** Adaptive thinking plus subagent orchestration handle most multistep reasoning internally; explicit chaining (sequential API calls) remains useful to inspect intermediate outputs or enforce a pipeline. The most common pattern is self-correction: draft → review against criteria → refine, each a separate call you can log, evaluate, or branch.

**Reduce file creation.** The models use temporary files (especially Python scripts) as scratchpads, which improves agentic coding outcomes. To keep trees clean: "If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task."

**Overeagerness.** Opus 4.5 and 4.6 tend to overengineer. The official minimal-solutions sample opens "Avoid over-engineering. Only make changes that are directly requested or clearly necessary," then sets four scopes: don't add features/refactors beyond the ask; don't document code you didn't change; don't add defensive handling for impossible scenarios (validate only at system boundaries); don't create abstractions for one-time operations or hypothetical futures. "The right amount of complexity is the minimum needed for the current task."

**Tests and hardcoding.** Claude can overfocus on making tests pass. Countersample: "Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds... Implement a solution that works correctly for all valid inputs, not just the test cases... Tests are there to verify correctness, not to define the solution. If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them."

**Minimizing hallucinations.** The official `<investigate_before_answering>` sample: "Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering... Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers."

## Capability-specific tips

**Vision.** Opus 4.5 and 4.6 improved image processing, data extraction, and multi-image contexts; this carries into computer use. Giving Claude a crop tool (letting it "zoom" into image regions) gives consistent uplift; Anthropic publishes a crop-tool recipe in the platform cookbook.

**Frontend design.** Without guidance the models converge on generic "AI slop" aesthetics. The official `<frontend_aesthetics>` sample pushes distinctive typography (no Arial/Inter/Roboto), committed color themes with CSS variables, high-impact motion (one well-orchestrated staggered page load over scattered micro-interactions), layered backgrounds over solid colors, and warns the model it still converges on common choices (e.g. Space Grotesk) across generations. Full text at the source URL, or the frontend-design skill in the claude-code repo.

## Migration considerations

Moving prompts from earlier generations to current models:

1. Be specific about desired behavior and output.
2. Add quality modifiers ("Include as many relevant features and interactions as possible. Go beyond the basics...").
3. Request animations and interactive elements explicitly.
4. Update thinking configuration to adaptive thinking + `effort` (no `budget_tokens`).
5. Migrate away from prefilled responses (see Output and formatting above).
6. Tune anti-laziness prompting down: 4.6+ models are proactive and overtrigger on guidance written for older, lazier models.

The migration guide at /docs/en/about-claude/models/migration-guide covers per-model steps, including Sonnet 4.5 → Sonnet 5 (effort default change, removal of manual extended thinking).
