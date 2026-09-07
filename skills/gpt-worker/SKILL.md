---
name: gpt-worker
description: >-
  Dispatch a headless OpenAI GPT worker (gpt-5.6-luna at max effort through
  `codex exec`) from plain Bash to do scoped, verifiable implementation: a
  spec'd multi-file edit, a reproduced bug, tests for a module. No Orca
  dependency; the visible-terminal path is the luna section of
  `orca-lite-dispatch`. Use this whenever the user explicitly asks to delegate
  coding to GPT, codex, or luna headlessly, or says "dispatch gpt", "gpt
  worker", "codex worker", "use codex", "run this on codex", or "delegate to
  gpt". Covers the launch recipe, the brief contract, reading the result files,
  and the mandatory code-verifier hand-off. Not for research or code review.
---

# GPT worker dispatch

`codex exec` is the headless GPT surface on this machine. The Claude Code Agent
tool cannot target GPT (its model enum is Claude-only), so the dispatch surface
is a Bash command run in the background, the same shape as `gemini-worker`.
The visible path, a codex terminal in Orca, lives in `orca-lite-dispatch`; the
`subagent-delegation` rule decides which surface a run gets.

## When to use

A scoped change with a way to tell it worked: a spec'd edit across named files,
a bug with a reproduction, tests to add. The brief carries a test command the
worker runs to check itself. Luna at max is a strong implementer, not a cheap
one; a trivial mechanical edit is better on `gemini-worker`. This is a worker,
never a reviewer.

Check `command -v codex` first. If it is absent, fall back to `clean-coder` and
say codex was unavailable.

## Launch recipe

Write the brief to a file, then run in the background through Bash. One
command, no wrapper script:

```
cd <workdir> && codex exec -m gpt-5.6-luna -c model_reasoning_effort=max \
  --dangerously-bypass-approvals-and-sandbox -C "$PWD" --json \
  -o <scratchpad>/gpt-<task>.md - < <scratchpad>/gpt-<task>-brief.md \
  > <scratchpad>/gpt-<task>.jsonl
```

Why each flag:

- `-m gpt-5.6-luna -c model_reasoning_effort=max`: effort is a config override,
  not a flag, the same shape as the luna argv in `orca-lite-dispatch`. Pin both
  even though the config defaults match: inside an Orca session `CODEX_HOME`
  points at Orca's per-account home, not `~/.codex`, so the config that loads
  depends on where the session runs. Model ids come from
  `$CODEX_HOME/models_cache.json`; trust it over any id written here.
- `--dangerously-bypass-approvals-and-sandbox`: parity with the visible luna
  path. The worker gets a full shell in the directory; the directory boundary
  lives in the brief and the verifier catches escapes. See the sandboxed
  alternative below.
- `-C "$PWD"`: the working root. Codex loads the repo's `AGENTS.md` from here.
- `--json`: one JSONL event per line on stdout, redirected to a file for
  counting tool calls or relaying progress. Nothing goes to stderr on a clean
  run, so an empty `.err` file means nothing.
- `-o <file>`: the worker's final message, the only thing the session reads
  back.
- `-` with stdin redirect: the brief comes from a file, so quoting is never a
  problem. Without a prompt argument codex waits on stdin, so a launch that
  omits both hangs silently.

There is no timeout flag and no `timeout` binary on this machine, so the
background run is the only bound. Check in on the `.jsonl` line count if a run
looks stuck.

The session does not edit files in `<workdir>` while the worker runs; reading is
fine. Run each parallel worker in its own directory; they must not share a
tree. Use a child worktree only when isolation is needed.

### Sandboxed alternative

Replace `--dangerously-bypass-approvals-and-sandbox` with `--approve-for-me`
when the workdir holds anything the worker must not touch. It runs the worker
in the workspace-write sandbox and routes any escalation request through
codex's automatic reviewer, so nothing blocks on a prompt. It completed the
same probe task as the bypass run, in eight shell calls, with no approval
stall. The two flags are mutually exclusive with `-s`; passing `-s
workspace-write` alongside it exits 2 before the run starts. The sandbox
blocks network by default, so a test command that installs packages or hits a
service needs the bypass flag.

## Brief contract

Codex reads `$CODEX_HOME/AGENTS.md` and the repo's `AGENTS.md`, so shared
conventions can live there. CLAUDE.md is still invisible, so anything that lives
only in CLAUDE.md must be restated in the brief. Every brief states:

- The absolute working directory on the first line.
- The test command to run (the oracle), and that the worker must run it before
  finishing.
- The files in scope.
- "Do not touch anything outside this directory." and "Do not commit." unless
  the session wants the commit.
- A reply cap under 100 words. The `-o` file is all the session reads back.

Write it the way `prompt-guides/references/openai-gpt-5p6.md` describes:
outcome, constraints, evidence, and completion bar, then room to choose the
path. Say what the run authorizes (luna is persistent and will keep going).
Leave out "be concise"; GPT-5.6 already is, and the cap above does the job.

## Reading the result

```
cat <scratchpad>/gpt-<task>.md
```

The exit code is the status: 0 on a completed turn, 2 on a bad argument. The
`.jsonl` file is the event stream: `thread.started` (with `thread_id`, usable
with `codex exec resume`), `turn.started`, `item.started` / `item.completed`
with `item.type` of `agent_message`, `command_execution` (carries `command`),
`file_change`, or `error`, and `turn.completed` with `usage` (`input_tokens`,
`cached_input_tokens`, `output_tokens`, `reasoning_output_tokens`). Count
`command_execution` items to gauge looping.

An `error` item at the top of the stream reading "loading hooks from both
hooks.json and config.toml" is Orca's codex home carrying both files; it is a
warning and the run proceeds.

## Hand-off

The worker's diff goes to a fresh-context `code-verifier` (opus, high) every
time; never let a codex run verify its own output. On a pass, the session
commits or merges per the git rules. On a fail there is no cheaper tier to
escalate from, since the run is already at max: report the findings and hold,
or re-run on the visible surface so the user can steer. This skill dispatches
and reads; it never commits.

## Pitfalls

- Effort travels in `-c model_reasoning_effort=max`, not a CLI flag.
- A trivial task still costs about 130k input tokens, almost all cache reads,
  because the codex home loads its plugins and skills on every run. Batch small
  edits into one brief rather than one run each.
- `--ignore-user-config` drops the model default, marketplaces, and MCP
  servers along with the hooks warning; leave it out.
- `--ephemeral` skips the session log when the run should leave no thread
  behind; without it, `codex exec resume <thread_id>` can continue a run.
- `codex update` can change flags; re-check `codex exec --help` after one.
