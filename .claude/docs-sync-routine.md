# Weekly docs sync: routine instructions

These are the instructions for the `claude-api-docs-sync` routine. The routine's own prompt just points here, so changes to how the sync works go through a normal PR on this file.

You maintain a Claude API knowledge base in this repo (markdown files under `docs/`). Each run is a weekly incremental update. Accuracy matters more than coverage: this is used by developers as a reference.

## Sources (official only, in this order)

1. Release notes: https://platform.claude.com/docs/en/release-notes/overview (append `.md` to any platform.claude.com docs URL to get clean markdown)
2. Models overview: https://platform.claude.com/docs/en/about-claude/models/overview
3. Any docs page a release note links to, for the details of that change
4. SDK changelogs: https://github.com/anthropics/anthropic-sdk-python/releases and https://github.com/anthropics/anthropic-sdk-typescript/releases

Do not rely on your own memory for model IDs, prices, limits or beta header names. Your training data may be out of date. Take every such fact from a page you fetched this run. Never hard-code a list of "current models" from memory: read the models overview every run and make `docs/MODELS.md` match it.

## Steps

1. Read `docs/README.md` and the top of `docs/CHANGELOG.md` to find the date of the last update. Also pick up any items an earlier entry marked as "left for the next scheduled run".
2. Fetch the release notes and list every entry dated after the last update.
3. Fetch the models overview and diff it against `docs/MODELS.md` (IDs, pricing, context, max output, default effort, knowledge cutoff, legacy/retired status). Fix any mismatch, even if no release note mentions it.
4. For each change, fetch the linked docs page and update the relevant file in `docs/` (or create a new one for a new feature area and add it to the index in `docs/README.md`). Include Python and TypeScript examples where useful, copied from or checked against the official page.
5. Prepend an entry to `docs/CHANGELOG.md`: date, a "Sources" line listing the exact URLs you fetched, one bullet per change (with the release-note date), and a "Files Modified" table. If nothing changed, add a short "No changes detected" entry listing the sources you checked.
6. Update the last-updated dates in `docs/README.md` for every file you touched.
7. If you could not fetch a page or aren't sure about a detail, say so in the changelog entry and in the file. Never guess.

## Delivering the update

- Create a branch named `docs-sync/YYYY-MM-DD` from the latest `main`. Do not push to `main`.
- Make one commit: `Weekly update - YYYY-MM-DD: <brief summary>`.
- Push the branch and open a pull request into `main`.
- PR title: `Weekly docs sync - YYYY-MM-DD: <brief summary>`. If the update includes a new model, a model deprecation or retirement, a price change, or a breaking API change, prefix the title with `[IMPORTANT] `.
- PR body: a short plain-English "What changed" list at the top (one line per change, most important first), then the source URLs, then the files modified. If nothing changed, still open the PR with just the changelog entry so there's a record that the run happened.
