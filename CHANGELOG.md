# Changelog

All notable changes to this repository are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.7.0] - 2026-11-05

### Added

- `prompt-guides`: references for Claude Opus 5.5, Sonnet 5.5, and Haiku 5.5,
  each condensed from its official prompting page. The router has a row for
  each, and the "Why the per-model files matter" section gains a bullet for
  each new file.

### Changed

- `prompt-guides`: the six existing Claude references were re-fetched on
  2026-11-05 and patched against the current pages. Best practices now
  covers the 5.5 models, the new per-model thinking defaults, and the
  medication-dosing system prompt blocks. Quotes that had been shortened
  are verbatim again. Claims the pages no longer make are gone, including
  the Fable 5.1 account-date cutoff and a Claude Code version floor in the
  Opus 5 file.
- `prompt-guides`: one routing note now covers all three pages written as
  changes from a predecessor (Fable 5.1, Opus 5.5, Sonnet 5.5). Where the
  newer page is silent, the predecessor's file applies.

## [0.6.1] - 2026-11-05

### Changed

- Agent profiles no longer list `codebase-memory-mcp` or `context7` tools,
  since neither server is installed now. `code-verifier`, `plan-architect`,
  `prose-writer`, and `sweep` lose the graph tools; `doc-researcher` and
  `plan-architect` lose context7. Body text that pointed at those tools now
  points at Grep, Glob, Read, and official docs, and `clean-coder` gets the
  same rewrite.
- `plan-architect`: `model: Fable` is now `model: fable`, since the docs only
  list lowercase aliases.

## [0.6.0] - 2026-09-07

### Added

- `gpt-worker` skill: dispatch a headless GPT worker (gpt-5.6-luna at max
  effort through `codex exec`) from plain Bash, no Orca dependency. Covers the
  launch recipe and why each flag exists, the sandboxed alternative, the brief
  contract (codex reads AGENTS.md, CLAUDE.md stays invisible), the JSONL event
  stream and result file, and the mandatory `code-verifier` hand-off. Fires on
  explicit request only.

### Changed

- `gemini-worker`: default tier raised from `gemini-3.8-flash-low` to
  `gemini-3.8-flash-medium`; escalation after a verifier fail is now
  `gemini-3.8-flash-high`. The visible argv in `orca-lite-dispatch` and the
  delegation rule follow.
- `subagent-delegation`: the Gemini and Orca sections merged into one
  cross-vendor workers section with a vendor table, and a new surface rule
  that picks headless or a visible Orca terminal per run: headless outside
  Orca; inside Orca, a terminal for implementation the user may want to watch,
  long or multi-file runs, and anything that may need steering, headless for
  short mechanical runs with a test oracle and for parallel fan-out, with word
  overrides and a tie to visible. The per-model section gains a Fable 5.1
  entry: background subagents while the session keeps working, and a one-line
  note of what was dispatched.
- `orca-lite-dispatch`: the luna section now points at `gpt-worker` for the
  headless path and the brief contract; description names the codex worker.
- README: agent profiles split into Claude profiles under `agents/` and a table
  of other-vendor workers with their headless and visible paths; new section
  summarising what the delegation rule decides.

## [0.5.0] - 2026-09-07

### Added

- `agents/`: the seven subagent profiles (`sweep`, `doc-researcher`,
  `page-capture`, `prose-writer`, `clean-coder`, `code-verifier`,
  `plan-architect`) moved in from `~/.claude/agents`, so the profiles the
  delegation rule names live next to the skills they use.
- `rules/subagent-delegation.md`: the delegation rule itself, moved in from
  `~/.claude/rules`. Both are symlinked back into `~/.claude`.

### Changed

- Repository renamed from `skills` to `agent-harness`; the README now describes
  the whole harness and how to link it into `~/.claude`.

## [0.4.0] - 2026-09-06

### Added

- `gemini-worker` skill: dispatch a headless Gemini 3.8 Flash worker through the
  `agy` CLI from plain Bash, no Orca dependency. Covers the launch recipe and
  why each flag exists, the brief contract (absolute workdir, test oracle, reply
  cap, directory boundary; CLAUDE.md is invisible to Gemini), reading the JSON
  envelope, and the mandatory `code-verifier` hand-off. Fires on explicit
  request only; defaults to `gemini-3.8-flash-low`, escalates to medium after a
  verifier fail.

### Changed

- `orca-lite-dispatch`: added a "Launch a gemini worker" section for the
  visible-terminal path: the interactive argv, the trust-prompt Enter, and the
  `tui-idle` gap that forces screen polling, pointing at `gemini-worker` for the
  brief. Description and pitfalls updated to name the Gemini worker.

## [0.3.1] - 2026-09-05

### Changed

- `orca-lite-dispatch`: re-checked every recipe against Orca 1.4.197 and its
  `skills get orca-cli` guide. The skill now resolves the `ORCA` executable the
  way the updated `orca-cli` stub does, reads the agent handle from
  `agentTerminalHandle` with `startupTerminal.handle` as the older fallback,
  and no longer claims `worker-start --effort` rejects `max`. The terminal
  launch path stays because `worker-start` needs a Task Run. Pitfalls gained
  the fallback shell a bare `worktree create` can leave and the `--focus` flag.

## [0.3.0] - 2026-09-05

### Added

- `prompt-guides`: reference for Claude Fable 5.1 and Mythos 5.1, condensed
  from the official prompting page.
- `prompt-guides`: reference for GPT-6 Astra, condensed from OpenAI's
  "Using GPT-6 Astra" page.

### Changed

- `prompt-guides`: each per-model reference now distills its own official
  page and nothing else. Headers state the page's scope in the page's words,
  and the router reads the per-model file before the vendor's generic guide.
  Cross-model comparisons live only in `SKILL.md`, each naming its source
  file.
- `prompt-guides`: all references re-fetched on 2026-09-05. Best practices
  gained the model-specific guidance table and Fable 5.1 notes, Opus 5 the
  harness-level subagent caps, Opus 4.8 and Sonnet 5 the computer-use and
  browser toolset identifiers, and the OpenAI generic guide now reports
  gpt-6-astra as the vendor's stated default. The skill's own default
  OpenAI target stays GPT-5.6, recorded in the router.

## [0.2.0] - 2026-08-20

### Added

- `orca-lite-dispatch` skill: dispatch visible Orca terminal workers (codex on
  gpt-5.6-luna at max effort, claude, or any custom agent argv) from a
  coordinating Claude session and relay messages between them by reading and
  writing their terminals. No task state, receipts, or inbox; workers default
  to the coordinator's own worktree, with a child worktree only when the run
  needs isolation.

### Changed

- Indexed the `orca-lite-dispatch` skill in the README.

## [0.1.0] - 2026-08-12

### Added

- `prompt-guides` skill: a router `SKILL.md` over condensed official
  prompting guides (Claude generic + Opus 5 / Sonnet 5 / Fable 5 / Opus 4.8,
  OpenAI generic + GPT-5.6), each reference traceable to its source URL with
  fetch date. Sources fetched live 2026-08-12.

### Changed

- Indexed the `prompt-guides` skill in the README.
