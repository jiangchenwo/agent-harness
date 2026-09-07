# agent-harness

A multi-agent workflow harness for Claude Code: the skills, agent profiles, and delegation rule that decide which model does which work. Each part is meant to be symlinked into `~/.claude` so one repo holds the whole setup.

| Directory | Contents | Link target |
|---|---|---|
| `skills/<name>/` | Portable skills, each with a `SKILL.md` entry point and, where needed, a `references/` directory loaded on demand. | `~/.claude/skills/<name>` |
| `agents/` | Claude subagent profiles that pin a model and effort per task type. The delegation rule dispatches to these by name. | `~/.claude/agents` |
| `rules/subagent-delegation.md` | The standing rule on when to delegate, which profile or vendor worker to pick, how to brief it, and whether a vendor worker runs headless or in a terminal. | `~/.claude/rules/subagent-delegation.md` |

## Skills

| Skill | What it's for |
|---|---|
| [gemini-worker](skills/gemini-worker/SKILL.md) | Dispatch a headless Google Gemini worker (Gemini 3.8 Flash through the `agy` CLI) from Bash for mechanical, verifiable implementation, with a mandatory code-verifier hand-off. Fires on explicit request only. |
| [gpt-worker](skills/gpt-worker/SKILL.md) | Dispatch a headless OpenAI GPT worker (gpt-5.6-luna at max effort through `codex exec`) from Bash for scoped implementation, with the same brief contract and code-verifier hand-off. Fires on explicit request only. |
| [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md) | The visible surface for both vendor workers and for claude: launch them in Orca terminals the user can watch, and relay messages between them by reading and writing those terminals. |
| [prompt-guides](skills/prompt-guides/SKILL.md) | Condensed official Anthropic and OpenAI prompt-engineering guides, generic and model-specific, for writing prompts another model or agent will execute. |

## Agent profiles

### Claude profiles (`agents/`)

Each profile is a Claude Code subagent definition with a pinned model and effort. The Agent tool dispatches to them by name.

| Profile | Model, effort | Use for |
|---|---|---|
| [sweep](agents/sweep.md) | sonnet, low | Mechanical volume work: sweeps, listings, format checks, eval runs. |
| [doc-researcher](agents/doc-researcher.md) | sonnet, medium | Doc and web research with cited findings. |
| [page-capture](agents/page-capture.md) | sonnet, medium | Browser capture, scraping, PDF fetching to disk. |
| [prose-writer](agents/prose-writer.md) | sonnet, medium | Long-form docs and artifact prose. |
| [clean-coder](agents/clean-coder.md) | opus, medium | Scoped implementation with a commit-ready diff. |
| [code-verifier](agents/code-verifier.md) | opus, high | Fresh-context verification of completed work. |
| [plan-architect](agents/plan-architect.md) | fable, high | Implementation planning, read-only. |

### Other-vendor workers

The Agent tool's model list is Claude-only, so non-Claude workers dispatch through a skill instead of a profile. Each has a headless path (a Bash command, result written to a file) and a visible path (an Orca terminal). Both fire on explicit request only, and every diff they produce passes `code-verifier` before it lands.

| Worker | CLI, model | Headless | Visible in Orca | Fires on |
|---|---|---|---|---|
| Gemini | `agy`, gemini-3.8-flash-low | [gemini-worker](skills/gemini-worker/SKILL.md) | [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md), gemini section | "dispatch gemini", "gemini worker" |
| GPT | `codex`, gpt-5.6-luna at max | [gpt-worker](skills/gpt-worker/SKILL.md) | [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md), luna section | "dispatch gpt", "codex worker", "dispatch luna" |

## Delegation rule

[`rules/subagent-delegation.md`](rules/subagent-delegation.md) loads into every session and decides four things:

- Which dispatch: a table maps task complexity to a profile: mechanical work to `sweep`, research to `doc-researcher`, verification to `code-verifier`, planning to `plan-architect`, implementation to `clean-coder`, and so on. A built-in agent type without a profile gets an explicit `model`, never an inherited one.
- How to brief: five parts in order: objective with its reason, output format and size cap, tool guidance, boundaries, inputs as paths. Every brief carries an output cap because the return lands in the session context.
- Which surface for a vendor worker: outside Orca, always headless. Inside Orca, a visible terminal for implementation the user may want to watch, long or multi-file runs, and anything that may need steering; headless for short mechanical runs with a test oracle and for parallel fan-out. Words like "watch", "terminal", "headless", and "background" override the default, and a tie goes to visible.
- Per-model skews: Fable delegates dependably, Opus 5 delegates too readily and gets capped, Opus 4.8 delegates too little and gets pushed to fan out.

## Install

```sh
ln -s "$PWD/agents" ~/.claude/agents
ln -s "$PWD/rules/subagent-delegation.md" ~/.claude/rules/subagent-delegation.md
for s in skills/*/; do ln -s "$PWD/$s" ~/.claude/skills/; done
```
