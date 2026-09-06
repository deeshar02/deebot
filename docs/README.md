# Docs

The project's wiki — everything worth knowing that isn't the code itself. Start
here and follow the links.

This page is the index. Every doc in this folder is listed below; if it isn't
listed, it may as well not exist.

## Index

*Nothing here yet — the first `/dev:build` that produces something worth
recording will add to this list.*

<!--
Keep the index grouped by section, newest last within each group:

### Guides
- [Adding a new source to ingest](guides/adding-a-source.md) — one line on what it covers

### Reference
- [How the provenance ledger works](reference/provenance-ledger.md) — one line

### Recipes
- [Re-run a failed scrape](recipes/rerun-failed-scrape.md) — one line
-->

---

## What goes where

Three sections, split by what you're doing when you reach for them. Put a page
in exactly one — if it seems to fit two, it's probably two pages.

| Folder | For | The test |
|---|---|---|
| `guides/` | Doing a task, start to finish | "How do I ___?" |
| `reference/` | Understanding how something works | "What *is* ___, and how does it behave?" |
| `recipes/` | A short, specific fix or one-off | "___ broke / I need to do ___ once" |

Names are lowercase with dashes: `adding-a-source.md`, not `Adding A Source.md`.

## What does *not* go here

This is the part that keeps the wiki readable. These have homes already:

- **What's done and what's next** → `.dadai/PROGRESS.md`
- **Why we chose X over Y** → the decision log in `.dadai/PROGRESS.md`
- **What we're building and why** → `.dadai/PRD.md`
- **How to build a specific thing, once** → the plan in `.dadai/plans/`
- **What changed and when** → git history

A doc that restates any of the above is duplication, and duplication is how a
wiki rots — two copies of a fact, one of them quietly going stale.

## The rule for adding to it

Before writing a new page, search the existing ones. Updating a page that's
already close is almost always right; a new page is for genuinely new ground.
Every new page gets a line in the index above, in the same commit.
