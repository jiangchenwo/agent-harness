---
name: gemini-worker
description: >-
  Dispatch a headless Google Gemini worker (Gemini 3.8 Flash through the `agy` /
  Antigravity CLI) from plain Bash to do mechanical, verifiable implementation:
  add tests to a module, fix a reproduced bug, apply a spec'd multi-file edit.
  Runs at a fraction of the Opus cost with no Orca dependency. Use this
  whenever the user explicitly asks to delegate coding to Gemini, agy, or Flash,
  or says "dispatch gemini", "run this on gemini", "gemini worker", "use a cheap
  worker", or "delegate to gemini". Covers the launch recipe, the brief
  contract, reading the JSON result, and the mandatory code-verifier hand-off.
  Not for research, code review, or anything needing judgment about the repo.
---

# Gemini worker dispatch

`agy` (Antigravity CLI) is the only working Gemini surface on this machine; the
`gemini` CLI is installed but has no auth. The Claude Code Agent tool cannot
target Gemini (its model enum is Claude-only), so the dispatch surface is a Bash
command run in the background, the same shape as the luna path in
`orca-lite-dispatch`.

## When to use

A well-specified mechanical change with a way to tell it worked: tests to add, a
bug with a reproduction, a spec'd edit across named files. The brief carries a
test command the worker runs to check itself. Gemini Flash is fast and cheap but
loops on ambiguity. If the task needs judgment about what the repo should do,
keep it in-session or use `clean-coder`. This is a worker, never a reviewer.

Check `command -v agy` first. If it is absent, fall back to `clean-coder` and
say Gemini was unavailable.

## Launch recipe

Run in the background through Bash. One command, no wrapper script:

```
cd <workdir> && agy -p "<brief>" --model gemini-3.8-flash-medium \
  --add-dir "$PWD" --output-format json --dangerously-skip-permissions \
  --print-timeout 10m > <scratchpad>/gemini-<task>.json
```

Why each flag:

- `--model gemini-3.8-flash-medium`: the default tier. Effort is baked into the
  model id; there is no effort flag. Escalate to `gemini-3.8-flash-high` only
  after a `code-verifier` fail on the medium run. `agy models` is the source of truth for ids; trust it
  over any hardcoded name here if they disagree.
- `--add-dir "$PWD"`: required in headless mode. Without it there is no active
  workspace: the model is not told its cwd and burns turns on `find`, `lsof`,
  `ps aux` locating itself, and no rules files load. With it, AGENTS.md and
  GEMINI.md load and are followed.
- `--output-format json`: one envelope on stdout (see below). Use `stream-json`
  instead when the session needs to count tool calls or relay progress.
- `--dangerously-skip-permissions`: required for an unattended run. The worker
  gets a full shell in the directory, the same posture as luna and `claude -p`.
  The directory boundary lives in the brief; the verifier catches escapes.
  (`--sandbox` exists for terminal restrictions but is untested; leave it out of
  v1.)
- `--print-timeout 10m`: the only cap on Flash's looping. Raise it for a large
  suite that takes longer than ten minutes to run.

The session does not edit files in `<workdir>` while the worker runs; reading is
fine. Run each parallel worker in its own directory; four at once is fine, but
they must not share a tree. Use a child worktree only when isolation is needed.

## Brief contract

CLAUDE.md is invisible to Gemini, so the brief is the only rules channel that
works everywhere. Every brief states:

- The absolute working directory on the first line.
- The test command to run (the oracle), and that the worker must run it before
  finishing.
- The files in scope.
- "Do not touch anything outside this directory."
- A reply cap under 100 words. The JSON `response` is all the session reads
  back, so a long reply wastes tokens.

Any convention that lives only in CLAUDE.md must be restated in the brief, or put
in an AGENTS.md the repo already carries.

## Reading the result

The `json` envelope has `response`, `status`, `duration_seconds`, and `usage`
(input, output, thinking, cache read). Pull the fields that matter:

```
python3 -c 'import json,sys; d=json.load(open(sys.argv[1])); \
print(d["status"], d["duration_seconds"]); print(d["response"])' <scratchpad>/gemini-<task>.json
```

Check `status` before trusting the diff: a quota or timeout failure shows there,
not in `response`. `stream-json` emits NDJSON `init` / `step_update` / `result`
events instead; tool calls are `step_update` with `step_type: "tool"` and
`tool_name`, for counting loops or relaying progress.

## Hand-off

The worker's diff goes to a fresh-context `code-verifier` (opus, high) every
time; never let a Gemini run verify its own output. On a pass, the session
commits or merges per the git rules; on a fail, report the findings and hold, or
re-run once at `gemini-3.8-flash-high`. This skill dispatches and reads; it
never commits.

## Pitfalls

- Effort is part of the model id, not a flag: `gemini-3.8-flash-medium`, not a
  separate `--effort`.
- `agy update` can change flags; re-check `agy --help` after an update, and
  trust `agy models` over any id written here.
- Flash low is loopy (50+ shell calls on a small file, mostly re-running tests).
  The `--print-timeout` is the only guard; on a large repo that can mean repeated
  full-suite runs.
- Auth is the Google account, not an API key, so a run can fail on quota rather
  than logic. That surfaces in the JSON `status` field; recognise it there
  before falling back.
- Two upstream bugs were open at last check: #76 (stdout dropped on pipes; did
  not reproduce here in 13 runs, but on empty stdout re-run reading the envelope
  from a file) and #548 (headless ignores `permissions.allow`, the reason
  `--dangerously-skip-permissions` is required rather than an allowlist).
