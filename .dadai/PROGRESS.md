# Progress

> **Maintenance protocol:** read this file at the start of every session, and
> update it before ending any session in which code changed or a decision was
> made. `PRD.md` is the contract; this file is the status — what's done is
> answered here, not by reading code and inferring.
>
> **Markers:** `[ ]` not started · `[-]` in progress · `[x]` complete (check
> ran and passed) · `[!]` blocked (reason on the same line)

## Status

- **Current phase:** Phase 1 — Module 0: the three builders
  (`/meta:command-build`, `/meta:agent-build`, `/meta:skill-build`).
  This is the only committed scope; everything else is parked (PRD §10).
- **Last updated:** 2026-09-06
- **Next action:** Run the namespace spike (Module 0 task 0 below). It is a
  hard prerequisite — the builders scaffold into whichever layout the spike
  confirms — and it depends on nothing and nobody.
- **Waiting on the user:** nothing. Module 0 was scoped precisely so that none
  of the twelve open questions in PRD §12 block it (PRD §6.6).

## Decision log

- **2026-09-06** — **Scope narrowed. Module 0 is now three commands and
  nothing else:** the command, agent, and skill builders. The rest of the
  meta-layer specified in PRD v0.1 — schema registry, generated types, the
  post-agent validator, run traces, the error taxonomy — is deferred to PRD
  §6.5 and revisited after Module 0 lands. Rationale: those pieces are
  contracts *between pipeline stages*, and no stage exists yet, so building
  them now means designing against imagined consumers. Everything from PRD §7
  onward is likewise reduced to a design sketch pending a Phase 2 review of
  the PRD.
- **2026-09-06** — Builder names follow the PRD §5 convention
  (`<subsystem>:<object>-<verb>`), so they are `/meta:command-build`,
  `/meta:agent-build`, `/meta:skill-build` — not the `/build-command` form
  used in PRD v0.1 §6, which contradicted the convention it was meant to
  enforce. Low-stakes and reversible; flagged rather than asked.
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

## Task board — Module 0 (the only active work)

### 0. Prerequisite

- [ ] **Namespace spike** — one throwaway plugin, one skill, confirm
      `<plugin>:<skill>` resolves on the installed Claude Code version.
      Record the result here before writing any builder (PRD §6.1). Decides
      whether the builders emit plugin layout or flat
      `.claude/commands/` + `.claude/agents/` files.

### 1. `/meta:command-build`

- [ ] Scaffolds markdown at the correct path with frontmatter (name,
      description, user-invocable flag)
- [ ] Emits the house body skeleton: Rules / Process / Output
- [ ] Declares which agents and skills the command may invoke (PRD §4)

### 2. `/meta:agent-build`

- [ ] Scaffolds markdown under the plugin's `agents/` with frontmatter
- [ ] Emits an explicit tool allowlist — narrowest set that does the job,
      never `*` by default
- [ ] Emits hand-written input-contract and output-contract blocks in the
      prompt body (registry-generated later, PRD §6.5)
- [ ] States the judgment boundary: the one call this agent makes, and what
      it must escalate

### 3. `/meta:skill-build`

- [ ] Scaffolds the thin `SKILL.md` interface wrapper
- [ ] Scaffolds a **runnable** script stub in `scripts/`: argparse CLI,
      `--json` flag, documented exit codes, non-zero exit on contract breach
- [ ] Ships a test fixture that runs the stub and asserts the `--json` shape
- [ ] Applies the registration threshold — declines to register a skill when
      a plain script will do (PRD §5)

### 4. Shared enforcement (all three)

- [ ] Naming refused, not warned, when it breaks
      `<subsystem>:<object>-<verb>` — with a conforming suggestion offered
- [ ] Upward calls refused (skill→agent, agent→command) per PRD §4
- [ ] No silent overwrite of an existing component; stop and report collision
- [ ] Every generated component added to a single catalog file

### 5. Acceptance

- [ ] **Self-hosting test:** `/meta:command-build` generates a working
      replacement for itself, and each other builder regenerates a working
      component of its kind. Runs, not inspected (PRD §6.4).
- [ ] Deliberately malformed name is refused by all three (tested on purpose)
- [ ] A scaffolded skill runs and its shipped test passes with no hand-editing
- [ ] `CLAUDE.md` Commands table filled in with whatever run/lint/test
      commands Module 0 actually creates — until then no `/dev:*` check can
      run at all

## Deferred — not being worked, revisit at the Phase 2 PRD review

Everything below is carried forward from PRD v0.1 and is **not active
scope**. It is kept for reference, not as a to-do list. PRD §7–§9 are design
sketches; PRD §10 Phase 2 is where this gets re-planned with the benefit of
having actually built Module 0.

### Deferred meta-layer (PRD §6.5)

- Versioned JSON Schema registry · generated Pydantic types ·
  schema-block injection · post-agent output validator and re-prompt loop ·
  retry budget · run IDs and structured step traces · the five-class error
  taxonomy · CI schema-compatibility check

### Deferred pillars (PRD §7)

- **Ingest** — source registry format, GH Actions path for public sources,
  local Playwright path for authenticated sources, robots/rate-limit/ToS
  enforcement, OCR path with confidence carried forward. Concrete source list
  and licensed-vendor connectors additionally blocked on PRD §12 #3 and #2.
- **Normalize** — boilerplate stripping, dedup, PDF table extraction, entity
  resolution, unit/currency normalization, as-of-date stamping.
- **Retrieve** — chunking, hybrid dense+BM25 with reranking, metadata
  filters, citation anchors. Embedding/vector-store choice additionally
  blocked on PRD §12 #1 and #5.
- **Synthesize** — style guide, critic pass against generic phrasing,
  deterministic numeric-citation validator. Voice exemplars and the
  deliverable-type ranking additionally blocked on PRD §12 #7 and #8.
- **Publish** — design token file and its four renderers (CSS custom
  properties, Chart.js, matplotlib, `pptxgenjs`), review gate. First
  deliverable target and NT brand assets additionally blocked on PRD §12 #4
  and #10.

## Blockers

**None on active work.** Module 0 touches no source, corpus, client content,
vendor data, or brand asset, so nothing in PRD §12 stands in its way
(PRD §6.6).

Still open, and still worth answering whenever convenient — they gate the
deferred pillar work above, not Phase 1:

- **MNPI / data residency (PRD §9, §12 #1).** Whether MNPI-tagged content can
  ever reach an agent-layer step remains unresolved. Conservative default
  stands: such records would be ingested and normalized but excluded from
  agent invocation by a deterministic filter. Highest-priority question for
  the *pillar* work.
- **Licensed vendor ToS (PRD §9, §12 #2).** No PitchBook/Bloomberg/
  Morningstar connector gets built until per-vendor sanctioned-access status
  is confirmed. Default-deny.
- **Source list, first deliverable target, voice exemplars, entity taxonomy,
  corpus size, brand assets, deliverable-type ranking, retention policy** —
  PRD §12 items #3, #4, #6, #7, #8, #9, #10, #11.

## Changelog

- **2026-09-06** — Scope narrowed to Module 0 = the three builders. `PRD.md`
  revised to v0.2: §6 rewritten around the builders alone with the rest of
  the meta-layer moved to a deferred table (§6.5); §7 and §8 marked as design
  sketches; §10 milestones and §11 metrics rewritten so Phase 1 is Module 0
  and Phase 2 is a re-plan of this document; §3 goals and §12 preamble
  adjusted to match. `PROGRESS.md` task board restructured around the three
  builders, with the pillar work moved to a deferred reference section. No
  code written.
- **2026-09-06** — `PRD.md` and `PROGRESS.md` drafted from an initial
  discovery conversation (deployment: local Mac + GitHub Actions; model:
  Claude via native Claude Code invocation; most pillar-specific inputs
  deferred by the user and marked as inline assumptions). No code written.
  Repo confirmed greenfield prior to this change.
