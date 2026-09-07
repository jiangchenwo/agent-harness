---
name: orca-lite-dispatch
description: >-
  Dispatch visible Orca terminal workers (codex on gpt-5.6-luna at max effort,
  claude, a Gemini 3.8 Flash worker through agy, or any custom agent argv) from a
  coordinating Claude session, and relay messages between them by reading and
  writing their terminals. Use when the user says "lite dispatch", "dispatch
  luna", "luna max worker", "spawn luna and claude workers", "watch a gemini
  worker", or wants multi-agent work running in terminals they can watch, without
  the orca-orchestrator task/receipt machinery. Use the
  `orchestration` skill instead for task DAGs, worker_done waits, decision
  gates, or durable task state; use `orca-cli` for full ownership handoffs.
---

# Orca lite dispatch

Every worker runs in a visible Orca terminal. The coordinating session is the only
message bus: it reads worker terminals, decides, and forwards. There is no task state,
no receipts, and no inbox. If the coordinator session dies, the workers keep running,
and any later session can resume coordination by re-listing terminals.

Resolve the `ORCA` executable the way the `orca-cli` skill describes, then read
`ORCA skills get orca-cli` for the exact, version-matched command surface. The recipes
below add only what that guide does not: the luna-max launch argv and the relay
protocol.

## Launch a luna-max codex worker

`worktree create --agent codex` cannot pass model or effort flags.
`orca orchestration worker-start --model --effort` can, but it needs a Task bound to
a Run, which is the machinery this skill leaves out. Launch through a plain terminal
instead.

Default to the current worktree: no new checkout, no new branch, and the
worker's diff lands where the coordinator already sits.

```text
ORCA terminal create --worktree active --title <task-name> \
  --command 'codex --dangerously-bypass-approvals-and-sandbox -m gpt-5.6-luna -c model_reasoning_effort=max' --json
ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms 60000 --json
ORCA terminal send --terminal <handle> --text "<brief>" --enter --json
```

While a worker runs in the coordinator's worktree, the coordinator must not
edit files there; reading is fine.

Create a child worktree only when the run needs isolation: two workers must
edit the same repo at once, the coordinator keeps editing while the worker
runs, the working tree holds unrelated uncommitted changes, or the user asks
for a branch. Then create the checkout first and point the terminal at it:

```text
ORCA worktree create --name <task-name> --repo id:<repoId> --parent-worktree active --json
ORCA terminal create --worktree id:<repoId>::<newWorktreePath> --title <task-name> \
  --command '<same codex argv>' --json
```

Pass `--no-parent` instead of `--parent-worktree active` only for a standalone
run unrelated to the caller.

A delegated run gets its verification and merge or commit from the
coordinating session, not from here. This skill stays mechanism only.

## Launch a claude worker

Same default, a terminal in the current worktree:

```text
ORCA terminal create --worktree active --title <task-name> --command 'claude' --json
ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms 60000 --json
ORCA terminal send --terminal <handle> --text "<brief>" --enter --json
```

Only when the run needs its own checkout (same tests as above):

```text
ORCA worktree create --name <task-name> --agent claude --prompt "<brief>" --json
```

Use `result.agentTerminalHandle` from the create response as the one and only
handle (older runtimes return only `startupTerminal.handle`); do not also
`terminal create` an agent in that worktree.

## Launch a gemini worker

A visible Gemini worker (Gemini 3.8 Flash through `agy`) when the user wants to
watch or steer it. The `gemini-worker` skill covers the headless Bash path and
owns the brief contract; this is only the terminal argv.

```text
ORCA terminal create --worktree active --title <task-name> \
  --command 'cd <abs workdir> && agy --dangerously-skip-permissions --model gemini-3.8-flash-medium' --json
ORCA terminal send --terminal <handle> --enter --json      # accepts the trust prompt
ORCA terminal send --terminal <handle> --text "<brief>" --enter --json
```

No `--add-dir` here: interactive mode makes the cwd the workspace once trusted.
That trust prompt ("Do you trust the contents of this project?") blocks the
first real prompt, so send a bare Enter before the brief. Effort lives in the
model id, not a flag. Write the brief to the `gemini-worker` contract (absolute
workdir, test command, files in scope, reply cap) because CLAUDE.md is invisible
to Gemini.

`terminal wait --for tui-idle` never reports idle for agy (it stays on
`codex-trust-workspace`), so poll `terminal read` until the screen stops
changing instead of waiting on idle.

## Brief contract

End every worker brief with these two rules so terminal reads stay cheap to interpret:

- "When finished, print a final line starting with `WORKER_DONE:` followed by a
  one-line result summary."
- "If you are blocked on information another agent has, print a line starting with
  `NEED:` describing exactly what you need, then wait."

## Relay loop

1. `ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms 300000 --json`
   on the worker expected to finish next, then
   `ORCA terminal read --terminal <handle> --json`.
2. A `WORKER_DONE:` line in the tail means the worker finished; collect details from
   the transcript or its worktree.
3. A `NEED:` line means the worker is blocked; forward the request to the other
   worker with `terminal send`, wait for its answer, and send the answer back.
4. Idle with neither marker means the agent is waiting on input; read the tail and
   answer it directly.

Alternate between workers; never block indefinitely on one while the other is idle.

## Pitfalls

- `--worktree active` is right only when launching into the coordinator's own
  worktree. For a worker in any other checkout, pass its explicit selector; a bare
  `current` or `active` resolves from the coordinator's process and targets the
  wrong tree.
- On `terminal_handle_stale`, reacquire the handle with `terminal list` and continue
  with the replacement only; never send to both the old and new handle.
- A bare `worktree create` (no `--agent`) may open a fallback shell before the later
  `terminal create` adds the codex worker. Send only to the agent handle; close the
  shell only after `terminal list` confirms it is unused.
- `terminal create` opens a visible tab without switching focus when it can, and
  falls back to a background handle otherwise. Pass `--focus` when the user wants
  the worker in front.
- `terminal read` before `terminal send` unless the next input is obvious; a prompt
  sent while the TUI is mid-render can be lost.
- Quote the codex `--command` argv exactly as shown; effort must travel inside
  `-c model_reasoning_effort=...`, not a CLI effort flag.
- `terminal wait --for tui-idle` never satisfies for an agy worker; it holds on
  `codex-trust-workspace` even after the trust prompt is gone. Poll `terminal
  read` until the screen stabilises instead.
