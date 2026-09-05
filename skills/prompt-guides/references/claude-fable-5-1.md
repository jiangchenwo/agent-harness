# Prompting Claude Fable 5.1 (official guide, condensed)

- Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1
- Fetched: 2026-09-05. Append `.md` to the URL for the raw Markdown source.
- Covers: Claude Fable 5.1 and Claude Mythos 5.1. The page is organized by observed symptom; existing Fable 5 prompts should perform well without changes, but a handful of behavioral differences are worth knowing about.

| Symptom | Section |
|---------|---------|
| Unsure which effort level to run, or latency and cost are higher than the task warrants | Consider all effort levels |
| Little or no text between tool calls | Ask for user-facing progress updates |
| One tool call per turn in agent loops | Batch independent tool calls in agent loops |
| Requests fail with `bound to a different conversation`, or harness edits earlier turns between requests | Keep the conversation history append-only |
| Prose runs long and dense | Writing density |
| Chat replies carry less structure than the content needs | Formatting in chat |
| Summaries reproduce source wording without marking it as a quotation | Quoting retrieved sources |
| Turn ends before the work is done, or the model asks permission for work already requested | Finish the whole task |
| Client-side compaction summaries drop constraints, decisions, or exact details | Tell the model what to preserve in compaction summaries |
| Unrequested fixes or extensions, or more committed test files than the task called for | Keep changes and tests to what the task asks for |
| Answers from memory instead of searching at low effort | Search triggering at low effort |
| Benign coding requests return `stop_reason: "refusal"` | Reduce safeguard false positives |
| Whole files rewritten for small changes | Prefer targeted edits over whole-file rewrites |
| Long deliverables at `xhigh` or `max` effort take a long time or hit `max_tokens` | Leave room for long outputs at xhigh and max effort |
| Lead agent idles while subagents run | Let the lead agent keep working while subagents run |
| Answers about charts and dense images miss detail | Give vision work tools to crop and zoom |

Fable 5.1 runs safety classifiers and can return `stop_reason: "refusal"`. See the "Reduce safeguard false positives" section and the Refusals, fallback, and billing page.

## Consider all effort levels

Effort is the primary control for trading off intelligence, latency, and cost. Start at `high` (the default), then test `low`, `medium`, `xhigh`, and `max` against your own evals. Effort level names do not correspond to the same amount of thinking across models, so re-run the sweep even if you already ran one on Fable 5.

Capability gains over Fable 5 appear across all effort levels and are largest at higher settings. At `medium`, results roughly match Fable 5 at lower cost. At `low`, Fable 5.1 is often competitive with Claude Opus and Sonnet models on cost per task while scoring higher, so include it in comparisons wherever you would otherwise run a smaller model at higher effort.

Two effort-specific behaviors have their own sections: at `low`, the model calls search and retrieval tools less often; at `xhigh` and `max`, it can think for longer before writing a long deliverable.

## Ask for user-facing progress updates

Fable 5.1 writes fewer user-facing updates during long tool-calling turns than Fable 5 does. The gap is more pronounced at higher effort and in longer tool chains.

First, check that your client receives progress updates at all. Short notes between tool calls come back as progress-update `thinking` blocks, which are empty under the default `thinking.display` of `"omitted"`. Set `display: "updates"` (beta, `thinking-display-updates-2026-08-18` header) and render each non-empty `thinking` block as a status line, or set `"summarized"` to receive them with summarized reasoning.

Second, audit your prompt for instructions that suppress narration, such as "hold all findings for the final response." Remove those lines before adding anything.

For more updates (pair programming, human-in-the-loop work):

> Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own — what you found, what you did, and what's next — so a reader who only sees the last message has the full picture.

If your product collapses or hides tool output, tell the model via a turn-scoped system message (`clear_at: "next_user_message"`, beta):

> Only you see that command's output — the user's terminal shows at most a few lines of it. If the user needs to read any of it, put it in your reply.

## Batch independent tool calls in agent loops

In coding and computer-use loops, Fable 5.1 may issue independent tool calls one per turn instead of in parallel. Answer quality is unaffected, but each extra turn costs tokens, a round trip, and wall-clock time. A one-sentence nudge addresses it:

> First privately list what you need next; then request every item that doesn't depend on another's result in this one response.

Append a fresh copy of this nudge after each set of tool results as a turn-scoped system message: a `role: "system"` entry in `messages` with `clear_at: "next_user_message"`. The beta header is `mid-conversation-system-clear-at-2026-08-21`. Leave earlier copies in the array byte-for-byte; the API clears them so the model reads only the newest copy and they cost no input tokens. Deleting or rewriting earlier entries restarts the prompt cache and invalidates thinking blocks that follow them.

Without the beta, place the sentence in a text block after the `tool_result` blocks in the same user message instead.

See the source page for per-language code samples showing the full loop structure.

## Keep the conversation history append-only

Append each assistant turn exactly as the API returned it, thinking blocks included, and do not edit earlier turns between requests. For accounts created on or after August 31, 2026, thinking blocks are valid only in the exact conversation that produced them: a request that replays a thinking block after its prefix (system prompt, tool list, or any earlier message) has changed returns a 400, or drops the affected blocks if you set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta, `thinking-binding-controls-2026-08-01` header). Future models are expected to enforce this check for all accounts.

The edits that trip the check are the same ones that restart the prompt cache: injecting and removing per-turn reminders, summarizing older turns in place, or changing the system prompt mid-session. Send per-turn reminders as turn-scoped system messages, change instructions or tools with a mid-conversation system message instead of rewriting `system` or `tools`, and let server-side compaction or context editing do any trimming.

For client-side compaction, the simplest shape is to replace the whole history with one summary message plus the new user turn and replay nothing else: no thinking blocks carry over, so nothing fails. Because cache reads are cheaper on Fable 5.1, compacting early to save cost may no longer be the right tradeoff, so experiment with later compaction points.

To find edits your harness already makes, run a session with `prefix_mismatch_behavior: "drop_block"` and log `input_transformations`, or capture exact requests over a few normal turns and confirm that consecutive requests are byte-identical up to appended turns.

## Writing density

Fable 5.1's prose is sometimes denser than Fable 5's: sentences run longer with fewer paragraph breaks. An instruction that names the anti-pattern helps:

> Mannered prose substitutes metaphor and flourish for direct statement. Instead of "a parameter worth varying," the mannered writer produces "a dial worth turning." Instead of "this point still matters," they write "this point earns its keep." The phrases exist to display the writer, not to convey the idea, and readers can tell. That is why mannered prose irritates: it makes the reader work harder so the writer can perform. It is also imprecise. Metaphors drag in connotations the writer did not choose and cannot control. The fix is to say what you mean. When a literal phrase is available, use it.

The short version also works:

> Please remove all mannered prose.

## Formatting in chat

Fable 5.1 uses bold less and is less likely to reach for headers, lists, or quotation marks in chat. Earlier anti-formatting rules (written when prior models overused bullets) can now suppress structure the content needs. Remove anti-formatting language or replace it with a rule about when specific formatting is appropriate:

> Use lists and bullet points when asked to, or when the content is multifaceted enough that they help with clarity. If the person explicitly requests minimal formatting, always format your responses without bullet points, headers, lists, or bold emphasis, as requested. In conversational, personal, or emotional exchanges, keep to plain prose.

## Quoting retrieved sources

Fable 5.1 is more likely than Fable 5 to reproduce passages of source text without marking them as quotations. Adding one complete example of a correct response to the system prompt addresses this: the user's request, the response, and a rationale sentence. The example in the source page shows the model using tool calls, synthesizing across two outlets in its own indirect speech, and quoting only sparingly with attribution. Replace the `[web_search: ...]` lines in the example with your own tool's name.

## Finish the whole task

On complex asynchronous workloads, Fable 5.1 can describe what it would do next instead of doing it, or stop to ask permission for a step the original request already covered. Two system prompt additions together mitigate this; apply both. If you need to limit prompt length, use only the first.

The first:

> You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking 'Want me to…?' or 'Shall I…?' will block the work. For reversible actions that follow from the original request, proceed without asking. Stop only for destructive actions or genuine scope changes the user must decide. Offering follow-ups after the task is done is fine; asking permission before doing the work is not.
>
> Exception: when the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one.
>
> Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ('I'll…', 'let me know when…'), do that work now with tool calls. That includes retrying after errors and gathering missing information yourself. Do not stop because the context or session is long. End your turn only when the task is complete or you are blocked on input only the user can provide.
>
> Before running a command that changes system state (such as restarts, deletes, or config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.

The opening sentence carries much of the effect; keep it as written. If your product needs the model to stop for specific confirmations, add a sentence after it listing them.

The second:

> # Delivering work
> The user's request — or the plan they approved — sets the scope, and the scope is the deliverable: don't quietly narrow, widen, or swap it. Read ambiguity the way a careful colleague would: make routine judgment calls yourself, and check in only when different readings would lead to materially different work. If you see a real problem with the task as specified, say so in a sentence or two and keep building under stated assumptions; if the user hears the concern and reaffirms, that is their decision, so deliver the full request.
>
> If a question comes up partway, first do everything that doesn't depend on the answer; then state the assumption you made, or — when going ahead on a wrong guess would be unsafe or would make the work useless — put the question at the end of a turn that also delivers that progress. If one part turns out to be blocked, complete every other part in full and say exactly what you left out and why — the whole task is the deliverable, and scaling it down is the user's call, not yours. A step you have decided on is something to run, not to announce: describing the next step and ending the turn leaves it undone until the user replies.
>
> Keep changes to what the request needs. Something else you notice worth doing — cleanup or documentation the task didn't call for, a change to a file the task didn't require — is a suggestion to make at the end, not a change to make; actions clearly beyond what the ask implies, and risky or destructive ones, still need the user's go-ahead.

## Tell the model what to preserve in compaction summaries

Server-side compaction already instructs the model on what to retain. For client-side compaction, use:

> Summarize the transcript inside \<summary\>\</summary\> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.

## Keep changes and tests to what the task asks for

When implementing open-ended features, Fable 5.1 may fix nearby code, extend behavior the task did not mention, or commit more test files than the change warrants. With the following instruction, unrequested additions and excess committed test code drop substantially with no measurable change in task success:

> If, while working or testing, you find a pre-existing bug, a performance concern, or behavior the task doesn't mention, don't fix, optimize or extend it in this change unless the requested behavior cannot work without it; report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and don't build for the other readings as well. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or this repository already keeps tests for this kind of change, sized like the neighboring test files — roughly one focused test per stated behavior — and don't turn scratch checks into additional permanent test files. This is about extras only: implement every behavior the task asks for, completely.

## Search triggering at low effort

At `low` effort, Fable 5.1 is less likely than Fable 5 to call a search or retrieval tool, and more likely to answer from memory. The simplest fix is to raise effort for the affected turns rather than the whole conversation (see the Change effort mid-conversation page).

For a prompt nudge instead:

> When a query centers on a name you do not confidently recognize, or recognize from a fast-moving area like AI models and developer tools where the landscape shifts within months, the name itself is the thing to verify: search before answering, and include the name as the user wrote it in at least one query alongside any reformulations. This holds even when you have some background on it — partial background is exactly what makes an out-of-date answer sound authoritative, so familiarity is not a reason to skip the search.

## Reduce safeguard false positives

Fable 5.1's safety classifiers produce fewer false positives than Fable 5's did at launch, and finding vulnerabilities in source code is permitted. False positives still occur and return `stop_reason: "refusal"`. Three situations make them more likely:

- Compile-check phrasing: ask "Are there any bugs in this program?" rather than "Does this program compile without errors?"
- Lesser-known programming languages: give the model context about what the language is and how it works, such as access to its documentation.
- Base64 in tool output: tools that return base64-encoded data into the model's context can trigger false positives; removing them is the recommended fix.

## Prefer targeted edits over whole-file rewrites

Fable 5.1 is more likely than Fable 5 to rewrite an entire file rather than make a targeted edit. The resulting file is usually the same, but a rewrite costs more output tokens and time unless the file is short or most of it is changing. Append to the system prompt or first user message:

> The number of tokens used to edit files is best minimized, all else being equal. Therefore, when it will not affect the end result, try to surgically edit a file rather than rewrite the entire thing.

## Leave room for long outputs at xhigh and max effort

At `xhigh` and especially `max`, Fable 5.1 can think for longer before writing its reply. When a request asks for a long deliverable, it may draft much of that deliverable in thinking and then write it out again as the reply. The simplest approach: run such requests at `high` and move to `xhigh` or `max` only where you have measured a quality gain.

If you do run them at `xhigh` or `max`, set `max_tokens` to leave room for both the thinking and the reply, not just the expected reply length. Then append to the user message, replacing `[max_tokens]` with the actual value (for example, 64,000):

> Everything produced in one reply, including any reasoning or drafting done before the reply, counts toward a single limit of about [max_tokens] tokens. If that limit is reached before the reply is finished, the person receives a cut-off response and has to start over. Composing an entire output or deliverable in full as reasoning and then again as a reply would double the length of the turn without improving the result, so don't do that.
>
> Instead, when the person has asked for a long or effort-intensive deliverable such as a multi-section document, a large table or dataset, or a complete code file, spend extra effort on understanding the request, checking the inputs the answer depends on, settling the structure and other difficult decisions, and otherwise using the reasoning space to reason and the output space to write an output. Usually it is not needed to draft an output multiple times.

## Let the lead agent keep working while subagents run

On coding tasks, letting the lead agent continue while subagents run lowers average time to completion at similar quality, token usage, and cost. To set this up: have the tool that starts a subagent return immediately; pass each subagent's result back to the lead in a later `user` message once it is ready; give the lead a separate tool it can call when it wants to wait for a result. The model still often chooses to wait; the time savings come from runs where it carries on with other work.

## Give vision work tools to crop and zoom

Fable 5.1 does its best work on complex visual inputs such as dense charts when it can iteratively analyze, crop, and verify what it sees. Run it as an agent with access to a container holding the raw images and basic image-processing libraries (PIL, OpenCV) pre-installed. If a container is too much overhead, an image-cropping tool alone delivers most of the uplift: a tool that returns a chosen region of the image, cropped and enlarged, lets the model examine specific details in depth and scales test-time compute with image tokens. See the crop tool recipe in the Anthropic cookbook.
