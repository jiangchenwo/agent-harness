# skills

Portable skills for AI agents. Each skill lives in `skills/<name>/` with a `SKILL.md` entry point and, where needed, a `references/` directory loaded on demand.

| Skill | What it's for |
|---|---|
| [gemini-worker](skills/gemini-worker/SKILL.md) | Dispatch a headless Google Gemini worker (Gemini 3.8 Flash through the `agy` CLI) from Bash for mechanical, verifiable implementation, with a mandatory code-verifier hand-off. Fires on explicit request only. |
| [orca-lite-dispatch](skills/orca-lite-dispatch/SKILL.md) | Dispatch visible Orca terminal workers (codex on gpt-5.6-luna at max effort, claude, a Gemini 3.8 Flash worker through agy, or any custom agent argv) from a coordinating Claude session, and relay messages between them by reading and writing their terminals. |
| [prompt-guides](skills/prompt-guides/SKILL.md) | Condensed official Anthropic and OpenAI prompt-engineering guides, generic and model-specific, for writing prompts another model or agent will execute. |
