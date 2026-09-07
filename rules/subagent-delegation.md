# Subagent delegation

Sessions here run on expensive models. Subagents exist to keep bulk reading,
searching, and mechanical work off the session model so its tokens go to
judgment. Delegate work that partitions cleanly: research, broad searches,
independent parallel tracks. Keep interdependent coding in-session.

## Choosing the dispatch

Match the dispatch to the task's complexity. Effort cannot be set in the
Agent call (the tool has a `model` parameter but no effort parameter), so
"choose the effort" means "dispatch to the named profile that pins it".
Profiles live in `~/.claude/agents/`.


| Complexity                                                                  | Dispatch                            |
| --------------------------------------------------------------------------- | ----------------------------------- |
| Mechanical: sweeps, listings, format checks, eval runs                      | `sweep` (sonnet, low)                |
| Mechanical implementation with a test oracle, on explicit request           | `gemini-worker` skill (agy, gemini-3.8-flash-low) |
| Scoped implementation on GPT, on explicit request                           | `gpt-worker` skill (codex, gpt-5.6-luna max) |
| Standard: doc and web research, comparisons                                 | `doc-researcher` (sonnet, medium)   |
| Judgment: verification of completed work                                    | `code-verifier` (opus, high)        |
| Judgment: implementation planning                                           | `plan-architect` (fable, high)      |
| Judgment: scoped implementation                                             | `clean-coder` (opus, medium)        |
| Writing: docs (including code-grounded docs with citations), artifact prose | `prose-writer` (sonnet 4.6, medium) |
| Browser capture, scraping, PDF fetching                                     | `page-capture` (sonnet, medium)     |
| Either worker in a terminal the user can watch, in Orca                     | `orca-lite-dispatch` skill          |


When a named profile fits, dispatch to it instead of general-purpose or
Explore. When none fits, set `model` explicitly in the call; never inherit by
omission — on a Fable session that silently runs the subagent on the most
expensive tier. That applies to every built-in type without a profile:
general-purpose, Explore, Plan, claude-code-guide, claude. Default to sonnet.

`opus` in the profiles resolves to Opus 4.8, pinned by
`ANTHROPIC_DEFAULT_OPUS_MODEL` in `settings.json`, even when the session runs
Opus 5. Same price; Opus 4.8 follows a brief literally and delegates less.

## Briefing

- Brief precisely: objective, output format, tool guidance, boundaries, and
the reason for the task. Vague briefs are the top documented failure.
- A brief has five parts, in this order: objective (one sentence, with the
reason the task exists), output format and size cap, tool guidance (which
tools, which to avoid), boundaries (paths and actions off limits), and
inputs (paths, not pasted content). Named profiles already carry the
per-model guidance, so a profile dispatch needs no skill read.
- Read the `prompt-guides` skill for the target model only when dispatching
general-purpose or Explore, or when the prompt will be reused.
- Give every brief an output-size cap; the subagent's return lands in the
session context.

## Running

- Scale the fan-out to the task: one agent for a lookup, several only for
genuinely complex work. Keep working while they run.
- Default to background; go foreground only when the very next action needs
the result.
- Retry and follow up by resuming the same agent (SendMessage), never by
respawning; resuming keeps its context, its open browser tab, and its warm
prompt cache.
- Bulk data stays on disk. A subagent that fetches pages, PDFs, model logs, or
any tool result over about 50 KB writes the payload to a file the brief
names and returns the path plus a summary. Never route page bytes,
screenshots, or base64 payloads through a model's context: read
`get_page_text` or `curl` output into a file, or have `javascript_tool`
write straight to disk.
- Probe before fan-out. Run the procedure on one item and check the output
before dispatching N agents with it. A fan-out that fails costs N times, and
the fix then goes to each agent by SendMessage, not by a fresh round.

## Per dispatching model

- **fable 5**: delegates dependably — use subagents freely. For long runs,
spawn fresh-context verifiers over self-critique.
- **opus 5**: delegates too readily — cap it. Only large, genuinely independent
tracks; never work finishable in a few tool calls, never to verify its own
output (it self-verifies).
- **sonnet 5**: no strong skew to correct — follow the general guidance above
and its `prompt-guides` entry.
- **opus 4.8**: delegates too little. Fan out independent tracks in the same
turn. Send model-load, generate, and eval commands with verbose stdout to one
`sweep` agent and resume it per run; keep only the scoring decision
in-session.
- Any other session model: read its entry in the `prompt-guides` skill.

## Cross-vendor workers

Non-Claude workers for implementation, on the user's explicit request only.
No standing tier; never propose one unprompted. Applies from every session
model; the per-model skews above still govern eagerness.

| Worker | CLI, model | Headless | Visible in Orca | Trigger words |
| ------ | ---------- | -------- | --------------- | ------------- |
| Gemini | `agy`, `gemini-3.8-flash-low`; medium only after a verifier fail | `gemini-worker` | `orca-lite-dispatch`, gemini section | "dispatch gemini", "gemini worker" |
| GPT    | `codex`, `gpt-5.6-luna` at max | `gpt-worker` | `orca-lite-dispatch`, luna section | "dispatch gpt", "codex worker", "dispatch luna", "use luna" |

- Availability: `command -v agy` or `command -v codex` at dispatch. Absent:
  fall to `clean-coder` and say the worker was unavailable.
- The Agent tool's model enum is Claude-only, so the skill is the dispatch
  surface for both.
- Verification: every worker diff passes a fresh-context `code-verifier`
  (opus, high) before it lands; a worker never verifies its own output. Pass:
  commit or merge per the git rules. Fail: report and hold.
- Rules gap: CLAUDE.md is invisible to both workers. Rules that must reach the
  worker go in the brief or an AGENTS.md the repo carries (codex reads
  AGENTS.md; Gemini reads AGENTS.md and GEMINI.md).

### Surface

Headless (Bash, result file only) or visible (an Orca terminal the user can
watch and steer). Both share the brief contract and the verifier hand-off;
only the launch argv differs.

- `ORCA_WORKTREE_ID` unset: headless, always.
- In Orca, visible when the run is implementation the user may want to watch
  or review as it happens, is long or touches several files, may need steering
  or a clarifying answer mid-run, or the user says "watch", "terminal",
  "visible", "spawn".
- In Orca, headless when the run is short and mechanical with a test oracle,
  is one of several parallel workers, only its diff and summary matter, or the
  user says "headless", "background", "quiet".
- Tie: visible. A terminal can be ignored; a headless run cannot be watched
  after the fact.
- Say which surface was chosen and why in one line at dispatch.

### In Orca

- A visible worker runs in the current worktree on the current branch; a child
  worktree and branch only when the run needs isolation. Mechanics live in the
  `orca-lite-dispatch` skill, including its sentinel contract for reading the
  terminal.
- In place: verify the working-tree diff, then commit. On a branch: merge
  `--no-ff` per the git rules on pass and delete the branch without asking; on
  fail, report and hold. Merge messages name no assistant or model, or the
  commit-msg guard stalls the automatic merge.
