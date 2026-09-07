# agent-harness

A multi-agent workflow harness for Claude Code: the skills, agent profiles, and delegation rule that decide which model does which work. Each part is meant to be symlinked into `~/.claude` so one repo holds the whole setup.

| Directory | Contents | Link target |
|---|---|---|
| `skills/<name>/` | Portable skills, each with a `SKILL.md` entry point and, where needed, a `references/` directory loaded on demand. | `~/.claude/skills/<name>` |
| `agents/` | Subagent profiles that pin a model and effort per task type. The delegation rule dispatches to these by name. | `~/.claude/agents` |
| `rules/subagent-delegation.md` | The standing rule on when to delegate, which profile to pick, and how to brief it. | `~/.claude/rules/subagent-delegation.md` |

## Skills

| Skill | What it's for |
|---|---|
| [gemini-worker](skills/gemini-worker/SKILL.md) | Dispatch a headless Google Gemini worker (Gemini 3.8 Flash through the `agy` CLI) from Bash for mechanical, verifiable implementation, with a mandatory code-verifier hand-off. Fires on explicit request only. |
| [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md) | Dispatch visible Orca terminal workers (codex on gpt-5.6-luna at max effort, claude, a Gemini 3.8 Flash worker through agy, or any custom agent argv) from a coordinating Claude session, and relay messages between them by reading and writing their terminals. |
| [prompt-guides](skills/prompt-guides/SKILL.md) | Condensed official Anthropic and OpenAI prompt-engineering guides, generic and model-specific, for writing prompts another model or agent will execute. |

## Agent profiles

| Profile | Model, effort | Use for |
|---|---|---|
| [sweep](agents/sweep.md) | sonnet, low | Mechanical volume work: sweeps, listings, format checks, eval runs. |
| [doc-researcher](agents/doc-researcher.md) | sonnet, medium | Doc and web research with cited findings. |
| [page-capture](agents/page-capture.md) | sonnet, medium | Browser capture, scraping, PDF fetching to disk. |
| [prose-writer](agents/prose-writer.md) | sonnet, medium | Long-form docs and artifact prose. |
| [clean-coder](agents/clean-coder.md) | opus, medium | Scoped implementation with a commit-ready diff. |
| [code-verifier](agents/code-verifier.md) | opus, high | Fresh-context verification of completed work. |
| [plan-architect](agents/plan-architect.md) | fable, high | Implementation planning, read-only. |

## Install

```sh
ln -s "$PWD/agents" ~/.claude/agents
ln -s "$PWD/rules/subagent-delegation.md" ~/.claude/rules/subagent-delegation.md
for s in skills/*/; do ln -s "$PWD/$s" ~/.claude/skills/; done
```
