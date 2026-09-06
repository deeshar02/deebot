# {Project Name} — {One-line descriptor}

> **Template usage:** This PRD is the contract for the project. Fill in every
> section before writing code. Keep it surgical — update it only when scope or
> architecture genuinely changes, not as a journal. The module list at the bottom
> drives `PROGRESS.md`, `/dev:plan`, and `/dev:build`.

## Vision

{2–4 sentences: what this product is, who it serves, and what "great" looks like.
Include the primary domain/URL if it's a web property, and the top non-functional
goals (e.g. must rank in search, must run offline, must handle X req/s).}

---

## Target Users

**Primary:** {Who they are, how they arrive, what they need to accomplish.}

**Secondary:** {Second audience and their distinct need.}

**Tertiary:** {Optional third audience.}

---

## Scope

### In Scope

- {Feature/surface 1}
- {Feature/surface 2}
- {Cross-cutting requirements: performance targets, SEO, accessibility, etc.}

### Out of Scope

- {Explicitly excluded feature — and why, if not obvious}
- {e.g. authentication, mobile app, i18n, CMS UI}

---

## Stack

| Layer | Current | Target |
|---|---|---|
| Build | {tool} | {tool, if migrating — else "unchanged"} |
| Framework | {framework} | {framework} |
| Styling | {approach} | {approach} |
| Data | {DB / files / API} | {target} |
| State | {state management} | {target} |
| Forms/Validation | {libs} | {target} |
| Hosting | {platform} | {target} |
| Package manager | {npm / pnpm / uv / cargo …} | {target} |
| Tests | {framework} | {target} |

*(Delete the "Target" column if there is no migration in flight.)*

---

## Constraints

- {Hard deployment/infra constraint}
- {Where content/data must live and why}
- {Compatibility requirements — URLs that must keep resolving, APIs that must stay stable}
- {Known debt that is acceptable vs. debt that must be paid down}

---

<!--
Module format — repeat for each module. Rules:
- Each module is a shippable increment, ordered by dependency.
- "Build" says what to do; "Delivers" says what is true when it's done.
- Number modules; sub-plans in docs/plans/ reuse these numbers (e.g. 3.1.slug.md).
- Every module here gets a matching checklist entry in PROGRESS.md.
-->

## Module 1: {Name}

**Build:** {What to implement — concrete enough that `/dev:plan` can turn it into
an executable plan without guessing. Name files/routes/systems where known.}

**Delivers:** {The observable end state — what works, what's measurable, what the
next module can rely on.}

---

## Module 2: {Name}

**Build:** {…}

**Delivers:** {…}
