---
name: prompt-guides
description: Condensed official prompt-engineering guides from Anthropic and OpenAI, generic and model-specific, for writing prompts that another model or agent will execute: subagent briefs, system prompts, delegated task specs, prompt migrations. Consult this whenever writing or reviewing a prompt for another agent, tuning a prompt for a specific target model (Claude Fable 5, Opus 5, Sonnet 5, Opus 4.8, GPT-5.6), or moving a prompt between models, even if the request never says "prompt engineering." Advisory: it supplements the host system's own rules, and where they conflict, the host wins.
---

# Prompt guides

Condensations of the official Anthropic and OpenAI prompt-engineering documentation, one file per guide, each traceable to its source page. Use them when writing a prompt some other model will run: the biggest prompt failures come from applying one model's conventions to another, and the official guides are where those differences are documented.

This skill is advisory. It supplements whatever rules the host system already has; if the host's conventions and these guides disagree, follow the host.

## How to use

1. Identify the target model the prompt is for.
2. Read the matching generic guide, plus the per-model file if one exists. Per-model files carry only the deltas, so the pair is the complete picture.
3. If the condensation isn't enough, every reference opens with its source URL; appending `.md` to it returns the raw Markdown of the full official page.

| Target model | Read |
|---|---|
| Any current Claude model | `references/claude-best-practices.md` |
| Claude Fable 5 or Mythos 5 | ...plus `references/claude-fable-5.md` |
| Claude Opus 5 | ...plus `references/claude-opus-5.md` |
| Claude Sonnet 5 | ...plus `references/claude-sonnet-5.md` |
| Claude Opus 4.8 | ...plus `references/claude-opus-4-8.md` |
| Any OpenAI model | `references/openai-prompt-engineering.md` |
| GPT-5.6 family | ...plus `references/openai-gpt-5p6.md` |
| Anything else | The vendor's generic guide only; skip the per-model files |

## Why the per-model files matter

Official guidance genuinely inverts between model versions, so a prompt tuned for one can degrade another:

- Claude Opus 5 over-verifies if you ask it to double-check; the general advice for other models is to ask for a self-check.
- Claude Opus 4.8 spawns fewer subagents and needs delegation encouraged; Opus 5 and Fable 5 delegate readily and may need it capped.
- Thinking is off by default on Claude Opus 4.8, on by default on Opus 5 and Sonnet 5, and always on for Fable 5.
- GPT-5.6 is more concise than GPT-5.5, so inherited "be concise" instructions can now make it too brief.

If the target model has no per-model file here, don't guess its deltas from a neighboring version. Use the generic guidance and, if it matters, fetch the vendor's current docs.

## Freshness

Each reference notes its source URL and fetch date; `references/sources.md` lists them all, plus which claims go stale fastest (thinking defaults, parameters that 400, effort recommendations, deprecation dates). When a claim here contradicts what an API actually returns, trust the API and re-fetch the source.
