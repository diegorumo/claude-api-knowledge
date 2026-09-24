# Claude API Knowledge Base

A markdown reference for building with the Claude API, kept current by a weekly Claude Code routine.

**Start here:** [docs/README.md](./docs/README.md) (full index) · [docs/MODELS.md](./docs/MODELS.md) · [docs/QUICK-REFERENCE.md](./docs/QUICK-REFERENCE.md) · [docs/CHANGELOG.md](./docs/CHANGELOG.md)

## How it stays up to date

- A scheduled routine (`claude-api-docs-sync`) runs every Monday at 10:00 UTC.
- It reads the official [release notes](https://platform.claude.com/docs/en/release-notes/overview) and [models overview](https://platform.claude.com/docs/en/about-claude/models/overview), updates the affected files in `docs/`, and appends an entry to `docs/CHANGELOG.md`.
- It opens a pull request instead of pushing to `main`, so each week's changes get a quick review before they land. PRs that include a new model, a price change, a deprecation or a breaking change are prefixed with `[IMPORTANT]`.

## Treat this as a summary, not the source of truth

This repo is written by an AI from Anthropic's public docs. Every change cites its source URL, but details can still be wrong or stale. Before relying on a price, limit, beta header or model ID, check the linked official page. The primary sources are:

- Release notes: https://platform.claude.com/docs/en/release-notes/overview
- Models overview: https://platform.claude.com/docs/en/about-claude/models/overview
- SDK releases: [anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python/releases), [anthropic-sdk-typescript](https://github.com/anthropics/anthropic-sdk-typescript/releases)
- Claude Code changelog: https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
