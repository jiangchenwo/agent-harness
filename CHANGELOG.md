# Changelog

All notable changes to this repository are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
