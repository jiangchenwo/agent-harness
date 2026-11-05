# Subagent delegation

Sessions here run on expensive models. Subagents exist to keep bulk reading,
searching, and mechanical work off the session model so its tokens go to
judgment. Delegate work that partitions cleanly: research, broad searches,
independent parallel tracks. Keep interdependent coding in-session.

## Choosing the dispatch

Match the dispatch to the task's complexity. Named profiles pin `model` and
`effort` in their frontmatter, so a dispatch to one passes neither. Pass
`effort` to a profile only when the user asks. Profiles live in
`~/.claude/agents/`.


| Complexity                                                                  | Dispatch                            |
| --------------------------------------------------------------------------- | ----------------------------------- |
| Mechanical: sweeps, listings, format checks, eval runs                      | `sweep` (sonnet, low)                |
| Mechanical implementation with a test oracle, on explicit request           | `gemini-worker` skill (agy, gemini-3.8-flash-medium) |
| Scoped implementation on GPT, on explicit request                           | `gpt-worker` skill (codex, gpt-5.6-luna max) |
| Standard: doc and web research, comparisons                                 | `doc-researcher` (sonnet, medium)   |
| Judgment: verification of completed work                                    | `code-verifier` (opus, high)        |
| Judgment: implementation planning                                           | `plan-architect` (fable, high)      |
| Judgment: scoped implementation                                             | `clean-coder` (opus, medium)        |
| Writing: docs (including code-grounded docs with citations), artifact prose | `prose-writer` (sonnet 4.6, medium) |
| Browser capture, scraping, PDF fetching                                     | `page-capture` (sonnet, medium)     |
| Either worker in a terminal the user can watch, in Orca                     | `orca-lite-dispatch` skill          |


When a named profile fits, dispatch to it instead of general-purpose or
Explore. When none fits, set both `model` and `effort` in the call; never
inherit by omission — on a Fable session that silently runs the subagent on
the most expensive tier. That applies to every built-in type without a
profile: general-purpose, Explore, Plan, claude-code-guide, claude. Default to
sonnet. Take effort from the table row the task matches: mechanical `low`,
standard `medium`, judgment `high`; `medium` when no row matches.

Model aliases track the newest model of each tier, and nothing pins them. As
of 2026-11: `opus` is Opus 5.5, `sonnet` is Sonnet 5.5, `haiku` is Haiku 5.5,
`fable` is Fable 5.1.

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

- **fable 5.1**: run subagents in the background and keep working while they
run; on coding tasks that lowers time to completion at similar quality,
token usage, and cost. Say in one line what was dispatched and where: 5.1
writes little text between tool calls, so a silent background dispatch is
invisible to the user.
- **sonnet 5.5**: at `xhigh` and `max` effort it may start its own review
rounds after finishing, sometimes with subagents. At those levels, once the
requested work is done and its checks pass, stop and report. Don't launch
reviewer subagents unless the user asked for a review; if a deeper review
seems worth it, say so at the end.
- Any other session model: read its entry in the `prompt-guides` skill.

## Cross-vendor workers

Non-Claude workers for implementation, on the user's explicit request only.
No standing tier; never propose one unprompted. Applies from every session
model; the per-model skews above still govern eagerness.

| Worker | CLI, model | Headless | Visible in Orca | Trigger words |
| ------ | ---------- | -------- | --------------- | ------------- |
| Gemini | `agy`, `gemini-3.8-flash-medium`; high only after a verifier fail | `gemini-worker` | `orca-lite-dispatch`, gemini section | "dispatch gemini", "gemini worker" |
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
