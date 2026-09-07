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
| Standard: doc and web research, comparisons                                 | `doc-researcher` (sonnet, medium)   |
| Judgment: verification of completed work                                    | `code-verifier` (opus, high)        |
| Judgment: implementation planning                                           | `plan-architect` (fable, high)      |
| Judgment: scoped implementation                                             | `clean-coder` (opus, medium)        |
| Writing: docs (including code-grounded docs with citations), artifact prose | `prose-writer` (sonnet 4.6, medium) |
| Browser capture, scraping, PDF fetching                                     | `page-capture` (sonnet, medium)     |
| Scoped implementation, on explicit request in Orca                          | `luna-max` via orca-lite-dispatch   |


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

## Gemini layer

A non-Claude worker for mechanical implementation with a test oracle, on the
user's explicit request only ("dispatch gemini", "gemini worker"). No standing
tier; never propose it unprompted. Applies from every session model. Mechanics
live in the `gemini-worker` skill.

- Availability: `command -v agy` at dispatch. Absent: fall to `clean-coder` and
  say Gemini was unavailable.
- Model: `gemini-3.8-flash-low`; escalate to `gemini-3.8-flash-medium` only after
  a `code-verifier` fail. Trust `agy models` for current ids.
- Verification: every Gemini diff passes a fresh-context `code-verifier` (opus,
  high) before it lands; a Gemini run never verifies its own output. Pass: commit
  or merge per the git rules. Fail: report and hold.
- Rules gap: CLAUDE.md is invisible to Gemini; rules that must reach the worker
  go in the brief or an AGENTS.md the repo carries.

## Orca layer

Active only when the session runs Orca-managed (`ORCA_WORKTREE_ID` is set); inert
everywhere else. Applies from every session model; the per-model skews above still
govern eagerness.

- `luna-max` (gpt-5.6-luna at max effort) substitutes for `clean-coder` on the
user's explicit trigger only — "dispatch luna", "use luna", "lite dispatch". No
standing volume tier, and never propose it unprompted.
- Mechanics defer to the `orca-lite-dispatch` skill: luna runs in the current
worktree by default, on the current branch; a child worktree and branch only
when the run needs isolation. The brief follows the `prompt-guides` luna entry
with the sentinel contract, relayed under supervision.
- Every luna diff passes `code-verifier` (opus, high) before it lands. In place:
verify the working-tree diff, then commit. On a branch: merge `--no-ff` per the
git rules on pass and delete the branch without asking; on fail, report the
findings and hold the branch.
- Merge messages follow the standard merge format and name no assistant or model,
or the commit-msg guard stalls the automatic merge.
- luna has no `agents/` profile: the Agent tool's model enum is Claude-only, so the
skill is its dispatch surface. A luna request outside Orca falls to `clean-coder`,
noting luna was unavailable.

