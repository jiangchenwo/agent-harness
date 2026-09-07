# Changelog

All notable changes to this repository are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
