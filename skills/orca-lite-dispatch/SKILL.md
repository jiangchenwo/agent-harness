---
name: orca-lite-dispatch
description: >-
  Dispatch visible Orca terminal workers (codex on gpt-5.6-luna at max effort,
  claude, or any custom agent argv) from a coordinating Claude session, and
  relay messages between them by reading and writing their terminals. Use when
  the user says "lite dispatch", "dispatch luna", "luna max worker", "spawn luna
  and claude workers", or wants multi-agent work running in terminals they can
  watch, without the orca-orchestrator task/receipt machinery. Use the
  `orchestration` skill instead for task DAGs, worker_done waits, decision
  gates, or durable task state; use `orca-cli` for full ownership handoffs.
---

# Orca lite dispatch

Every worker runs in a visible Orca terminal. The coordinating session is the only
message bus: it reads worker terminals, decides, and forwards. There is no task state,
no receipts, and no inbox. If the coordinator session dies, the workers keep running,
and any later session can resume coordination by re-listing terminals.

Read `orca skills get orca-cli` first for the exact, version-matched command surface.
The recipes below add only what that guide does not: the luna-max launch argv and the
relay protocol.

## Launch a luna-max codex worker

`worktree create --agent codex` cannot pass model or effort flags, and
`orca orchestration worker-start --effort` rejects `max`. Launch through a plain
terminal instead:

```text
ORCA worktree create --name <task-name> --no-parent --json
ORCA terminal create --worktree id:<repoId>::<newWorktreePath> --title <task-name> \
  --command 'codex --dangerously-bypass-approvals-and-sandbox -m gpt-5.6-luna -c model_reasoning_effort=max' --json
ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms 60000 --json
ORCA terminal send --terminal <handle> --text "<brief>" --enter --json
```

## Launch a claude worker

```text
ORCA worktree create --name <task-name> --agent claude --prompt "<brief>" --json
```

Use `startupTerminal.handle` from the create response as the one and only handle.

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

- Always pass an explicit `--worktree` selector. A bare `current` resolves from the
  coordinator's own process, not the worker's checkout, and targets the wrong tree.
- On `terminal_handle_stale`, reacquire the handle with `terminal list` and continue
  with the replacement only; never send to both the old and new handle.
- `terminal read` before `terminal send` unless the next input is obvious; a prompt
  sent while the TUI is mid-render can be lost.
- Quote the codex `--command` argv exactly as shown; effort must travel inside
  `-c model_reasoning_effort=...`, not a CLI effort flag.
