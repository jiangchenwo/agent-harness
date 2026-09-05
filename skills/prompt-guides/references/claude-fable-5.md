# Prompting Claude Fable 5 (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5
- Fetched: 2026-08-12. Append `.md` to the URL for the raw Markdown source.
- Covers: the prompting and scaffolding patterns specific to Claude Fable 5 and Claude Mythos 5, framed as behavioral differences from Claude Opus 4.8. The page defers capabilities, API changes, pricing, and availability to the launch page, and techniques that apply across all current Claude models to the best-practices page.

Claude Fable 5 takes on problems previously too complex, long-running, or ambiguous for prior models; it is particularly effective at end-to-end work that takes a person hours, days, or weeks. Teams see the best outcomes applying it to their hardest unsolved problems; testing only on simpler workloads undersells it. API notes: adaptive thinking only, summarized-only thinking output, no extended thinking budgets, and a `refusal` stop reason with fallback handling. Safety classifiers target offensive cybersecurity, biology/life sciences content, and extraction of summarized thinking; benign work in those areas can also trigger them, so configure server-side or client-side fallback to Claude Opus 4.8 to re-route declined requests.

## Capability improvements over Opus 4.8

Long-horizon autonomy (multiday goal-directed runs with strong instruction retention); first-shot correctness on complex well-specified problems; vision (dense technical images, screenshots, trained to use bash and crop tools on flipped/blurry/noisy images); enterprise workflows (financial analysis, spreadsheets, slides); code review and debugging (noticeably higher bug-finding recall, outside classifier-covered domains); navigating ambiguity; delegation (significantly more dependable at dispatching and sustaining parallel subagents). Not intended for offensive cybersecurity or biology/life sciences work; those can return `stop_reason: "refusal"`.

## Longer turns by default

Individual requests on hard tasks can run many minutes at higher effort; autonomous runs can extend for hours. Adjust client timeouts, streaming, and progress indicators before migrating; consider checking on runs asynchronously (scheduled jobs) rather than blocking. To prevent overplanning on ambiguous tasks:

> When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.

## Consider all effort levels

Effort is the primary intelligence/latency/cost control. `high` is the default for most tasks; `xhigh` for the most capability-sensitive workloads; `medium`/`low` for routine work. Lower effort on Fable 5 still performs well and often exceeds `xhigh` on prior models. At higher effort on routine work the model can gather context and deliberate beyond need; to prevent unrequested tidying or refactoring:

> Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well... Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.

## Strong instruction following

A brief instruction steers most behaviors, with no need to enumerate each by name. Example brevity instruction (instead of listing every verbosity pattern):

> Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did you find": the thing the user would ask for if they said "just give me the TLDR." Supporting detail and reasoning come after. Being readable and being concise are different things, and readability matters more. The way to keep output short is to be selective about what you include (drop details that don't change what the reader would do next), not to compress the writing into fragments, abbreviations, arrow chains like A → B → fails, or jargon.

Checkpoint behavior, same principle:

> Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.

## Ground progress claims during long runs

Instructing an audit against tool results nearly eliminated fabricated status reports in Anthropic's testing, even on tasks designed to elicit them:

> Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.

## State the boundaries

Fable 5 can occasionally take unrequested actions (drafting an unasked-for email, defensive git-branch backups). Define constraints:

> When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action.

## Parallel subagents

Fable 5 dispatches parallel subagents more readily than prior models. Use them frequently, say when delegation is appropriate, and prefer asynchronous orchestrator-subagent communication over blocking on each return. Long-lived subagents that keep context across subtasks save time and cost through cache reads. Sample: "Delegate independent subtasks to subagents and keep working while they run. Intervene if a subagent goes off track or is missing relevant context."

## Construct a memory system

Fable 5 performs particularly well when it can record lessons across runs. A Markdown file suffices: "Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong." Bootstrap from history: "Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]."

## Rare cases of early stopping

Deep into long sessions the model can end a turn with a statement of intent ("I'll now run X") without the tool call, or ask permission it doesn't need. A "continue" suffices interactively. For autonomous pipelines:

> You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task... For reversible actions that follow from the original request, proceed without asking... Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ("I'll…", "let me know when…"), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.

## Rare cases of context-budget concern

In very long sessions the model can suggest a new session or trim its own work, most often when the harness shows it a remaining-token countdown. Avoid surfacing explicit context-budget counts. If you must: "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work."

## Give the reason, not only the request

Fable 5 performs better when it understands intent: context lets it connect the task to relevant information rather than inferring intent on its own. Template: "I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request]."

## Readability when communicating with the user

In extended agentic conversations the model can produce hard-to-follow text: arrow-chain shorthand, deep implementation detail, references to thinking the user never saw. The official addendum distinguishes working notes from the final summary: terse shorthand is fine between tool calls; the final summary is for a reader who saw none of it, so it opens with the outcome, uses complete sentences, spells out terms, avoids arrow chains and invented labels, and gives each file, commit, or flag its own plain-language clause. "If you have to choose between short and clear, choose clear."

## Create a send-to-user tool

For long asynchronous agents, add a client-side tool that displays a message to the user verbatim without ending the turn (deliverables, progress updates with specific numbers, direct replies mid-loop). Tool inputs are never summarized, so content arrives intact. Schema: one required string property (`message`); render the input directly in the UI, return a simple acknowledgement. Defining the tool is not sufficient; without system-prompt elicitation Fable 5 rarely calls it:

> Between tool calls, when you have content the user must read verbatim (a partial deliverable, a direct answer to their question), call the send_to_user tool with that content. Use send_to_user only for user-facing content, not for narration or reasoning.

## Recommended scaffolding changes

- **Start at the top of your difficulty range.** Pick a task harder than what you'd assign prior models; have Fable 5 scope it, ask clarifying questions, and execute.
- **Make self-verification explicit in long-run prompts.** Separate fresh-context verifier subagents tend to outperform self-critique: "Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification."
- **Refactor existing prompts and skills.** Skills written for prior models are often too prescriptive for Fable 5 and can degrade output quality; remove older instructions if default performance is better.
- **Don't instruct Claude to reproduce its reasoning in the response.** Prompts that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category, causing elevated fallbacks to Opus 4.8. Audit skills and system prompts for show-your-thinking instructions when migrating. For reasoning visibility, read the structured `thinking` blocks from adaptive thinking, and surface progress through a send-to-user tool.
