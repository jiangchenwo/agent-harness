# Sources

Every reference file in this directory is a condensation of one official documentation page, fetched live on the date shown. Nothing was written from model memory.

| Reference file | Source URL | Fetched | Status | Bytes |
|---|---|---|---|---|
| claude-best-practices.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices | 2026-09-05 | 200 | 64,218 |
| claude-fable-5-1.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1 | 2026-09-05 | 200 | 54,572 |
| claude-fable-5.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5 | 2026-09-05 | 200 | 19,059 |
| claude-opus-5.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5 | 2026-09-05 | 200 | 12,465 |
| claude-sonnet-5.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5 | 2026-09-05 | 200 | 16,479 |
| claude-opus-4-8.md | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8 | 2026-09-05 | 200 | 16,595 |
| openai-prompt-engineering.md | https://developers.openai.com/api/docs/guides/prompt-engineering | 2026-09-05 | 200 | 37,374 |
| openai-gpt-6-astra.md | https://developers.openai.com/api/docs/guides/latest-model | 2026-09-05 | 200 | 16,564 |
| openai-gpt-5p6.md | https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6 | 2026-09-05 | 200 | 17,273 |

## Fetching the originals

Appending `.md` to any of these URLs returns the raw Markdown source on both platform.claude.com and developers.openai.com. This is the cheapest way to read the full page or refresh a condensation.

## Staleness

The claims most likely to go stale, and most likely to be acted on: thinking defaults per model, which API parameters return 400 errors, effort-level recommendations, refusal categories, beta header names, account-date cutoffs for API behavior changes, computer-use toolset identifiers, and deprecation dates. Re-check the sources when a new model ships or when a claim here contradicts what an API returns.

## Refresh history

- 2026-08-12: first fetch of seven pages.
- 2026-09-05: all pages re-fetched. The Claude pages and the OpenAI generic page had changed; the GPT-5.6 page was byte-identical. Two pages condensed for the first time: Claude Fable 5.1 and OpenAI's "Using GPT-6 Astra".
