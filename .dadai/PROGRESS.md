# Progress

> **Maintenance protocol:** read this file at the start of every session, and
> update it before ending any session in which code changed or a decision was
> made. `PRD.md` is the contract; this file is the status — what's done is
> answered here, not by reading code and inferring.
>
> **Markers:** `[ ]` not started · `[-]` in progress · `[x]` complete (check
> ran and passed) · `[!]` blocked (reason on the same line)

## Status

- **Current phase:** Pre-Phase 1 — PRD drafted, awaiting confirmation of open
  questions before the Phase 1 verification spike starts.
- **Last updated:** 2026-09-06
- **Next action:** Get answers to the 12 open questions in `PRD.md` §12
  (highest priority: #1 MNPI/data-residency, #3 initial source list, #4 first
  deliverable target). In parallel, the plugin/namespace verification spike
  (§5) can start now since it doesn't depend on any open question.

## Decision log

- **2026-09-06** — Deployment target confirmed as local Mac (interactive
  Claude Code sessions, authenticated ingest) + GitHub Actions (scheduled,
  unauthenticated ingest only, no browser session available). Drives the
  ingest architecture split in PRD §7.1.
- **2026-09-06** — Model/reasoning layer confirmed as Claude, invoked natively
  through Claude Code command/agent execution — no separate LLM API client
  needed in `src/` for the reasoning layer itself. Flagged as a distinct,
  still-open question: whether any GitHub Actions step will ever need
  agent-layer reasoning, which would require an Anthropic API key as a GH
  secret (not yet needed — Actions is scoped to deterministic skill-layer
  work only per §7.1).
- **2026-09-06** — Compliance/data-residency and most pillar-specific inputs
  (source list, corpus size, voice exemplars, entity taxonomy, brand assets,
  first deliverable target) were explicitly deferred by the user ("don't
  worry about the rest," "I will point you to the existing code when the
  time is right"). PRD.md written with `[ASSUMPTION: …]` markers in place of
  each, per the source process's rule against inventing figures or lists.
  These are not resolved — see PRD §12.
- **2026-09-06** — Repo confirmed genuinely greenfield: CLAUDE.md, PRD.md,
  PROGRESS.md, README.md were all still unfilled templates from the initial
  scaffolding commit before this session.

## Task board

### Module 0 — meta-layer

- [ ] Plugin/namespace verification spike: one throwaway plugin, one skill,
      confirm `<plugin>:<skill>` resolves on the installed Claude Code
      version (PRD §5) — **blocks nothing else, do this first**
- [ ] JSON Schema registry scaffolded (versioned, semantic versioning)
- [ ] Pydantic type generation from schema
- [ ] Schema-block injection into agent/command markdown
- [ ] Skill-layer `--json` contract enforcement (validate on exit)
- [ ] Agent-layer deterministic validator + re-prompt-on-failure loop
- [ ] `/build-command`, `/build-agent`, `/build-skill` scaffolders
- [ ] Self-hosting test: `/build-command` generates a working replacement for
      itself
- [ ] Run ID + structured step trace (observability)
- [ ] Error taxonomy implemented for at least: source-unreachable,
      schema-violation (PRD §6 table)

### Ingest

- [ ] Source registry format defined (config-driven, not per-source code)
- [ ] GitHub Actions workflow for unauthenticated/public sources
- [ ] Local on-demand path for authenticated/paywalled sources
      (Playwright + `storageState`)
- [ ] robots.txt / rate-limit / user-agent / ToS-record enforcement
- [ ] OCR path for scanned-image PDFs with confidence score carried forward
- [!] Concrete source list — **blocked on PRD §12 item #3**
- [!] Licensed-vendor connectors (PitchBook/Bloomberg/Morningstar) —
      **blocked on PRD §9 / §12 item #2, off-limits by default until
      resolved**

### Normalize

- [ ] Boilerplate/nav-chrome stripping
- [ ] Deduplication on normalized text
- [ ] PDF table extraction to structured form
- [ ] Entity resolution (empty seed table + fuzzy match + confirm-above-
      threshold flow)
- [ ] Unit/currency normalization
- [ ] As-of-date stamping + source/capture-timestamp pointer on every record

### Retrieve

- [!] Embedding model / vector store choice — **blocked on PRD §12 item #1
      (data residency) and #5**
- [ ] Chunking strategy
- [ ] Hybrid retrieval (dense + BM25) with reranking
- [ ] Metadata filters: asset class, manager, geography, doc type, as-of
      date, source confidence
- [ ] Citation anchors (page/paragraph) on every retrieval result

### Synthesize

- [ ] Institutional-finance style guide (generic v1)
- [!] Few-shot voice exemplars — **blocked on PRD §12 item #7**
- [ ] Critic pass rejecting generic/AI-sounding phrasing
- [ ] Deterministic numeric-citation validator (blocks unsourced claims)
- [!] Confirmed top-3 90-day deliverable types — **blocked on PRD §12 item
      #8, currently assumed as memo / competitive-intel brief / one deck**

### Publish

- [!] First deliverable target identified — **blocked on PRD §12 item #4,
      user will point to existing HTML/Chart.js deck or PPTX chartbook**
- [ ] Design token file (color, type scale, spacing, chart palettes, table
      styles)
- [ ] Token → CSS custom properties
- [ ] Token → Chart.js theme
- [ ] Token → matplotlib stylesheet
- [ ] Token → `pptxgenjs` theme
- [!] NT-approved brand assets — **blocked on PRD §12 item #10, placeholder
      palette in use until brand review**
- [ ] Review gate (draft + citation trail surfaced together, nothing
      publishes without it)

## Blockers

- **MNPI / data residency (PRD §9, §12 #1).** Whether MNPI-tagged content can
  ever reach an agent-layer step (and therefore Anthropic's API) is
  unresolved. Conservative default in place: MNPI-tagged records are
  ingested/normalized but excluded from agent invocation by a deterministic
  filter until this is answered. This is the single highest-priority
  blocker in the project.
- **Licensed vendor ToS (PRD §9, §12 #2).** PitchBook/Bloomberg/Morningstar
  connectors are not being built until per-vendor sanctioned-access status
  is confirmed.
- **Source list, first deliverable target, voice exemplars, entity
  taxonomy, corpus size, brand assets, deliverable-type ranking, retention
  policy** — all deferred inputs, tracked as PRD §12 items #3, #4, #6, #7,
  #8, #9, #10, #11. None block the Module 0 verification spike; all block
  the pillar work items marked `[!]` above.

## Changelog

- **2026-09-06** — `PRD.md` and `PROGRESS.md` drafted from an initial
  discovery conversation (deployment: local Mac + GitHub Actions; model:
  Claude via native Claude Code invocation; most pillar-specific inputs
  deferred by the user and marked as inline assumptions). No code written.
  Repo confirmed greenfield prior to this change.
