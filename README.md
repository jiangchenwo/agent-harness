# skills

Portable skills for AI agents. Each skill lives in `skills/<name>/` with a `SKILL.md` entry point and, where needed, a `references/` directory loaded on demand.

| Skill | What it's for |
|---|---|
| [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md) | Dispatch visible Orca terminal workers (codex on gpt-5.6-luna at max effort, claude, or any custom agent argv) from a coordinating Claude session, and relay messages between them by reading and writing their terminals. |
| [prompt-guides](skills/prompt-guides/SKILL.md) | Condensed official Anthropic and OpenAI prompt-engineering guides, generic and model-specific, for writing prompts another model or agent will execute. |
