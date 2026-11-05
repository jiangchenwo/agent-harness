---
name: prompt-guides
description: >-
  Condensed official prompt-engineering guides from Anthropic and OpenAI, generic and model-specific, for writing prompts that another model or agent will execute: subagent briefs, system prompts, delegated task specs, prompt migrations. Consult this whenever writing or reviewing a prompt for another agent, tuning a prompt for a specific target model (Claude Fable 5.1, Fable 5, Opus 5.5, Opus 5, Sonnet 5.5, Sonnet 5, Haiku 5.5, Opus 4.8, GPT-6 Astra, GPT-5.6), or moving a prompt between models, even if the request never says "prompt engineering." Advisory: it supplements the host system's own rules, and where they conflict, the host wins.
---

# Prompt guides

Condensations of the official Anthropic and OpenAI prompt-engineering documentation, one file per guide, each traceable to its source page. Use them when writing a prompt some other model will run: the biggest prompt failures come from applying one model's conventions to another, and the official guides are where those differences are documented.

This skill is advisory. It supplements whatever rules the host system already has; if the host's conventions and these guides disagree, follow the host.

## How to use

1. Identify the target model the prompt is for.
2. Read the per-model file first if one exists. Each is a distillation of that model's official page and nothing else. Then read the vendor's generic guide for the techniques the per-model page defers to it.
3. If the condensation isn't enough, every reference opens with its source URL; appending `.md` to it returns the raw Markdown of the full official page.

| Target model | Read first | Then |
|---|---|---|
| Claude Fable 5.1 or Mythos 5.1 | `references/claude-fable-5-1.md` | `references/claude-best-practices.md` |
| Claude Fable 5 or Mythos 5 | `references/claude-fable-5.md` | `references/claude-best-practices.md` |
| Claude Opus 5.5 | `references/claude-opus-5-5.md` | `references/claude-best-practices.md` |
| Claude Opus 5 | `references/claude-opus-5.md` | `references/claude-best-practices.md` |
| Claude Sonnet 5.5 | `references/claude-sonnet-5-5.md` | `references/claude-best-practices.md` |
| Claude Sonnet 5 | `references/claude-sonnet-5.md` | `references/claude-best-practices.md` |
| Claude Opus 4.8 | `references/claude-opus-4-8.md` | `references/claude-best-practices.md` |
| Claude Haiku 5.5 | `references/claude-haiku-5-5.md` | `references/claude-best-practices.md` |
| Any other current Claude model | `references/claude-best-practices.md` | |
| GPT-6 Astra, when named | `references/openai-gpt-6-astra.md` | `references/openai-prompt-engineering.md` |
| GPT-5.6 family, or an OpenAI target left unspecified | `references/openai-gpt-5p6.md` | `references/openai-prompt-engineering.md` |
| Any other named OpenAI model | `references/openai-prompt-engineering.md` | |
| Anything else | The vendor's generic guide only | |

Two routing notes:

- Three pages cover only what changed from a predecessor: Fable 5.1 from Fable 5, Opus 5.5 from Opus 5, and Sonnet 5.5 from Sonnet 5. Where the newer page is silent (for Fable 5.1, the memory system, send-to-user tool, and scaffolding changes the Fable 5 page describes), the predecessor file still applies. Haiku 5.5 has no such fallback: its page compares against Haiku 4.5, which has no file here.
- GPT-5.6 is this skill's default OpenAI target. The OpenAI generic guide now names gpt-6-astra as the vendor's default; read the GPT-6 Astra file when that model is actually the target.

## Why the per-model files matter

Official guidance genuinely inverts between model versions, so a prompt tuned for one can degrade another. Each claim below comes from the file named in parentheses:

- Claude Opus 5 over-verifies if you ask it to double-check; the general advice for other models is to ask for a self-check (`claude-opus-5.md`, `claude-best-practices.md`).
- Claude Opus 4.8 spawns fewer subagents and needs delegation encouraged (`claude-opus-4-8.md`); Opus 5 and Fable 5 delegate readily and may need it capped (`claude-opus-5.md`, `claude-fable-5.md`).
- Thinking is off by default on Claude Opus 4.8 (`claude-opus-4-8.md`), on by default on Opus 5 and Sonnet 5 (`claude-opus-5.md`, `claude-sonnet-5.md`), adaptive-only on Fable 5 (`claude-fable-5.md`), and always on for Opus 5.5 (`claude-opus-5-5.md`). Sonnet 5.5 can go no lower than `between_tools` (`claude-sonnet-5-5.md`).
- Claude Fable 5.1 writes fewer progress updates and less structure in chat than Fable 5, so anti-narration and anti-markdown rules written for earlier models now suppress output the reader needs (`claude-fable-5-1.md`).
- Claude Fable 5.1 rejects conversation history edited before a thinking block; harnesses that rewrote earlier turns or summarized in place for other models must move those changes to mid-conversation system messages (`claude-fable-5-1.md`). The best-practices page extends this to Claude 5.1 and later models (`claude-best-practices.md`).
- Claude Opus 5.5 returns a 400 error for `thinking: {"type": "disabled"}`, which Opus 5 accepted at `high` effort or below. Remove any instruction that asked for written-out reasoning in place of thinking (`claude-opus-5-5.md`).
- Claude Opus 5.5 ends some turns with a text-only progress update; an unattended loop that treats `end_turn` as completion stops early (`claude-opus-5-5.md`).
- Claude Sonnet 5.5 returns a 400 error for `between_tools` at `xhigh` or `max` effort, and effort steers its initiative: it checks in before finishing at `low` and `medium` and can do more than asked at higher effort (`claude-sonnet-5-5.md`).
- Claude Haiku 5.5 replaces `budget_tokens` with effort; at `low`, in long agent prompts, it is more likely to skip a search, stop early, or skip a check (`claude-haiku-5-5.md`).
- GPT-5.6 is more concise than GPT-5.5, so inherited "be concise" instructions can now make it too brief (`openai-gpt-5p6.md`).
- GPT-6 Astra asks for clarification more often than earlier OpenAI models and reacts to conflicting guidance in skill files, so prompts must state instruction priority and bias toward action (`openai-gpt-6-astra.md`).

If the target model has no per-model file here, don't guess its behavior from a neighboring version. Use the generic guidance and, if it matters, fetch the vendor's current docs.

## Freshness

Each reference notes its source URL and fetch date; `references/sources.md` lists them all, plus which claims go stale fastest (thinking defaults, parameters that 400, effort recommendations, deprecation dates). When a claim here contradicts what an API actually returns, trust the API and re-fetch the source.
