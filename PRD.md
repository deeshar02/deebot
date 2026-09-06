# Deebot — an agentic research operating system for institutional investment work

> **Contract status:** Draft v0.1, written from a short discovery conversation, not
> a full requirements interview. Several inputs the source process asks for
> (named source list, corpus size, licensed-vendor access, entity taxonomy, voice
> exemplars, brand assets, hard compliance constraints, the specific first
> deliverable) were not supplied — each is marked inline as
> `[ASSUMPTION: …]` where it materially shapes a decision, and rolled up in
> §12. Confirm or correct those before Phase 1 work goes past the verification
> spike.

## 1. Problem statement

The user is Head of Multi-Asset Solutions at Northern Trust Asset Management —
a CFA charterholder and FINRA-licensed investment professional whose work spans
client-facing investment leadership, product development, competitive
intelligence, and business development across institutional and intermediary
channels, concentrated in private markets, OCIO, and model portfolios. She is
technically fluent — she builds her own Python optimizers and large HTML/
Chart.js decks — so the constraint on her output is not capability, it is time.

Her research-to-publication workflow today is almost entirely manual. Source
material arrives from dozens of blogs, asset-manager and consultant websites,
regulatory filings, fund documents, and PDFs. She reads each one, extracts the
figures and claims that matter, reconciles them against what she already knows,
and then rebuilds essentially the same analysis three times over: once as a
memo, once as a deck, once as a newsletter or LinkedIn post. Nothing produced
in step one is reused in step two. Nothing is indexed, so nothing from three
months ago is retrievable except by memory or by re-reading the original
source. Every deliverable starts from a blank page, even when 80% of its
content — the underlying facts, figures, and citations — was already
assembled for a previous one.

`[ASSUMPTION: no per-deliverable time figures were supplied — the process this
PRD follows asks for hours-per-deliverable quantification, and none should be
invented. Directionally: a research memo or competitive-intel brief that draws
on 5–15 sources plausibly costs multiple hours of reading and reconciliation
before any writing starts, and that cost is paid again, from scratch, for each
of the three output surfaces. Replace this paragraph with real numbers once
she provides them — they matter for §11's success metrics.]`

The cost of the status quo is threefold: (a) research time is spent re-doing
extraction and reconciliation that a system could do once and reuse, (b) work
product has no institutional memory — a figure verified in March has to be
re-verified in June because it isn't indexed anywhere queryable, and (c) the
three output surfaces (memo, deck, newsletter/LinkedIn) are maintained as
independent artifacts with no shared source of truth, so a correction made in
one doesn't propagate to the others.

## 2. Users and usage contexts

**Primary user.** The user herself, working solo from a local Mac. She
initiates ingestion (scheduled and on-demand), reviews and corrects what the
system extracts, drives synthesis of memos and briefs, and is the sole
approver before anything publishes. Every design decision in this document
optimizes for her as a single expert operator, not for a multi-seat product.

**Secondary users.** `[ASSUMPTION: not confirmed — plausible secondary
consumers are colleagues on the Multi-Asset Solutions team who might receive
finished decks/memos, or a research associate who could eventually help
triage ingestion. The system should not architecturally foreclose this (see
§9's access-control note), but nothing in Phase 1–3 should be built
specifically for a second seat until she asks for one.]`

**Tertiary / gating role.** Whoever performs compliance or reviewer sign-off
before a deliverable reaches a client or the public — today, most likely the
user herself acting as her own reviewer, with the review-gate mechanism (§9)
designed so a formal compliance reviewer could be inserted later without
rearchitecting the publish pipeline.

## 3. Goals and non-goals

**Goals**

- Collapse ingest → normalize → retrieve → synthesize → publish into a
  pipeline where each stage's output is a durable, inspectable file — not a
  chat transcript — that the next stage and any future stage can consume.
- Make every fact that reaches a published deliverable traceable to a specific
  source document, capture timestamp, and location within it (page or
  paragraph anchor), so "where did this number come from" is always
  answerable in one lookup, not a re-read.
- Let a memo, a deck, and a newsletter about the same underlying research draw
  from the same normalized corpus and the same verified figures, so a
  correction made once is available to all three surfaces.
- Produce output in the user's own analytical voice — prose-forward,
  argumentative, institutional-finance register — not generic AI house style.
- Build the meta-layer (contracts, builders, observability, error taxonomy)
  once, first, so every pillar built afterward conforms to the same
  conventions instead of each pillar inventing its own.

**Non-goals**

- **Not a multi-tenant product.** This is a single-operator system for one
  named user on her own machine. It is not being built to onboard other users,
  support role-based access control across a team, or serve as a SaaS
  offering. If that changes, it is a new set of requirements, not an
  extension of this one.
- **Not a general-purpose web scraper or crawler.** Ingest targets a
  deliberately bounded, named set of sources she actually reads (§7.1) — it is
  not a discovery engine that finds new sources on its own, and it does not
  attempt to be comprehensive over the open web.
- **Not a replacement for her judgment on credibility, framing, or house
  view.** Retrieval surfaces sources with metadata and confidence signals;
  synthesis drafts arguments; neither is authorized to publish a final
  editorial position without her review (§9's review gate is non-negotiable,
  not a v2 nicety).
- **Not a trading, portfolio-construction, or order-routing system.** Nothing
  here reads or writes live positions, executes trades, or touches any system
  of record for client assets. It only reads public/licensed research
  content and produces documents.
- **Not a compliance-approval system.** It can enforce that a review gate
  exists and that citations are present; it cannot and does not substitute
  for the firm's actual compliance review process, and it must not be
  described to anyone as having done so.
- **Not attempting real-time or streaming ingestion.** Sources are polled on
  a schedule or pulled on demand; there is no requirement for sub-minute
  freshness anywhere in this system.
- **Not building bespoke connectors for licensed data vendors ahead of
  confirming ToS.** PitchBook, Bloomberg, Morningstar-class integrations are
  explicitly out of scope for automated ingestion until their terms are
  reviewed (§9) — this project will not quietly build a scraper against a
  vendor whose ToS forbids it.

## 4. Architecture overview

The system is organized as four layers, ordered by determinism and by who is
allowed to call whom. Calls point downward only; dependencies point inward
only. A skill never invokes an agent. An agent never invokes a command.

```mermaid
flowchart TD
    H["Human"] -->|invokes| C["Commands (.claude/commands or plugin skills)\nUse cases. Deterministic control flow.\nOwn orchestration and error handling."]
    C -->|"invokes (1+ per step)"| A["Agents (.claude/agents)\nReasoning layer. Non-deterministic.\nJudgment: what to extract, what's credible,\nhow to reconcile conflicts. May cache."]
    A -->|invokes| S["Skills\nDeterministic layer. Single-purpose,\nside-effect-explicit. Thin instruction\nwrapper over a script in scripts/."]
    S -->|calls| SS["Subsystems (src/)\nOrdinary libraries: auth/credential\nbroker, browser (Playwright), storage,\nprovenance ledger, observability."]

    style A fill:#fff3cd,stroke:#b08d00
    style C fill:#d4edda,stroke:#2d6a2d
    style S fill:#d4edda,stroke:#2d6a2d
    style SS fill:#d4edda,stroke:#2d6a2d
```

The inversion worth stating explicitly: the middle layer (agents) is the
non-deterministic one, sandwiched between a deterministic caller (commands)
and a deterministic callee (skills). Both boundaries need enforced contracts
as a result — a command cannot assume an agent's output is well-formed just
because the prompt asked for it, and a skill's CLI contract is the only thing
standing between "the agent decided to call this" and "the side effect that
actually happened." §6 specifies enforcement for both boundaries separately.
Any cache at the agent layer must key on **input + prompt version + model
identifier** — caching on input alone will silently serve reasoning from a
stale prompt or a retired model indefinitely.

The five-stage pipeline is orthogonal to this layering — each stage is
implemented using command → agent → skill → subsystem calls, and each stage's
output is a durable artifact the next stage reads from disk (or from the
corpus store), never from the prior stage's live context:

```mermaid
flowchart LR
    I["Ingest\nraw capture +\nprovenance"] --> N["Normalize\nclean, deduped,\nentity-resolved records"]
    N --> R["Retrieve\nindexed corpus,\nRAG-queryable"]
    R --> S["Synthesize\ncited draft\n(memo/brief/deck copy)"]
    S --> P["Publish\nreviewed HTML/PPTX/\nnewsletter artifact"]

    I -.->|"provenance ledger\n(source, timestamp, ToS record)"| Ledger[("Provenance\nledger")]
    N -.-> Ledger
    R -.-> Ledger
    S -.-> Ledger
    P -.-> Ledger
```

Each arrow in the pipeline diagram is a durable-artifact handoff, not a
function call — normalize does not run inside ingest's process, it reads
ingest's output. This is what makes each stage independently inspectable,
re-runnable, and debuggable in isolation, and it's why the observability
requirement in §6 records a step trace per stage rather than one opaque trace
for the whole run.

## 5. Packaging and namespacing

`[ASSUMPTION: the source process for this PRD asks that plugin-vs-command
behavior be verified against current Claude Code docs and empirically
confirmed before scaling, because the artifact model has moved recently and
documented nesting behavior has not always matched reported behavior. That
verification has not been run yet — it is Phase 1's first task (§10), not a
settled fact. Everything in this section is the target design, contingent on
that spike confirming `<plugin>:<skill>` resolution actually works on the
installed Claude Code version.]`

The namespacing unit is the plugin: a directory with a
`.claude-plugin/plugin.json` manifest holding `skills/`, `agents/`, and hooks,
whose skills are invoked as `<plugin>:<skill>`. This namespace can't collide
with skills defined at other levels, which is the property flat
`.claude/commands/` files don't reliably give once nesting is involved.

**One subsystem, one plugin.** Ingest, normalize, retrieve, synthesize,
publish, and meta each become their own plugin in a local marketplace repo,
versioned and installed independently. Adding a sixth pillar later means
adding a seventh plugin, not deepening a directory tree — that's the scaling
property this model is chosen for.

**Naming convention.** `<subsystem>:<object>-<verb>` — object before verb,
enforced consistently so the catalog stays guessable: `/ingest:source-scrape`,
`/normalize:table-extract`, `/retrieve:corpus-query`, `/publish:deck-render`,
`/meta:skill-build`. The three builders in Module 0 (§6) are the enforcement
point — they should refuse to scaffold a name that doesn't fit the pattern
rather than relying on convention alone.

**User-facing vs. internal is a frontmatter property, not a location.**
Orchestration entry points are marked user-invocable; internal machinery
(a normalize sub-step, a validator an agent calls) is marked not
user-invocable. This keeps the `/`-command surface small and legible as the
component count grows, independent of how many files live under the hood.

**The skill-vs-script registration threshold.** Every registered skill's
description occupies context on every request — a growing catalog is a
standing token cost paid whether or not that skill is used this turn. Skills
are the *interface*; scripts in `scripts/` are the *implementation*. An agent
invoking a Python CLI through Bash costs nothing at rest; the same logic
registered as a skill costs tokens forever. The threshold: **register a skill
only when an agent needs to discover it without being told which one to use**
— i.e., when the choice of tool is itself part of the judgment being
delegated to the agent. If a command step already knows exactly which script
to run and just needs an agent to supply arguments or interpret output,
that's a direct script call, not a skill registration. Concretely: something
like `pdf-table-extract` (one clear job, always the right tool when a PDF has
a table) is a script an agent shells out to; something like
`source-credibility-assess` (the agent must decide *whether* and *how* to
invoke judgment-bearing logic, and other agents may need to discover it too)
is a registered skill.

**Verification step (Phase 1, first task).** Build one throwaway plugin with
one skill, confirm it resolves as `<plugin>:<skill>` on the installed Claude
Code version, and record the result in `PROGRESS.md` before generating any
other component against this assumption.

## 6. Module 0 — the meta-layer

Module 0 ships before any pillar, and is validated by building one real
vertical slice through it (Phase 1's thin slice, §10) — not by inspection.
Meta-tooling expands to fill available time; the discipline here is to ship
the thinnest version of each piece below that the first end-to-end
deliverable actually needs, and let friction from that build drive the next
iteration.

**Typed contracts.** JSON Schema is the single source of truth for every
inter-stage payload (an ingest record, a normalized record, a retrieval
result, a synthesis draft, a publish manifest), held in a versioned registry
under semantic versioning. From each schema, two consumers are generated:
runtime types for the script layer (Pydantic, since the skill layer is
Python-first per §7's chosen approach) and a schema block injected into the
markdown of whichever agent or command must honor it, so the contract is
visible in the prompt, not just enforced after the fact. A breaking change is
detected by a registry-level compatibility check run in CI (GitHub Actions,
per the deployment answer) that diffs a schema against its previous published
version and fails the build on a non-additive change without a version bump;
the required response to a genuine breaking change is a new major schema
version plus an explicit migration note in that schema's changelog, not a
silent overwrite.

**Contract enforcement, per layer.** A skill is a real process with a real
exit code, so `--json` on every skill genuinely enforces its output contract:
validate against the schema on exit, fail loudly (non-zero exit, structured
error to stderr) on mismatch. Agents and commands are prompts, not processes,
and can't enforce a contract on themselves — so enforcement sits in a
deterministic validator script that runs after every agent invocation,
checks the returned payload against the schema, and on failure re-prompts the
agent with the specific validation error rather than passing malformed data
upward. Retry budget: `[ASSUMPTION: 2 re-prompt attempts, 3 total tries —
not confirmed with the user. On exhaustion, the run halts and escalates per
the "halt" behavior in the error taxonomy below, rather than passing
best-effort malformed output downstream.]`

**The three builders.** `/build-command`, `/build-agent`, and `/build-skill`
scaffold new components against every convention above: naming (§5),
directory layout, frontmatter (user-invocable or not), input/output schema
stubs wired to the registry, the executable stub for skills (a real script in
`scripts/` with a CLI, per §5's threshold), a test fixture, and a registry
entry. Self-hosting is the acceptance test: `/build-command` must be able to
generate a working replacement for itself. If it can't, the convention it
encodes is underspecified, not the builder.

**Observability.** Every command execution gets a run ID and a structured
step trace: which agents fired, which skills they called, what the cache did
(hit/miss/stale-key-evicted), and where wall-clock time went, per pipeline
stage. This is written to the provenance/run store as the run happens, not
reconstructed after — a chain of non-deterministic agent calls is undebuggable
without a trace that exists independent of whether the run succeeded.

**Error taxonomy.** Five failure classes, with a required behavior each:

| Class | Example | Required behavior |
|---|---|---|
| Source unreachable | Site down, DNS failure, timeout | Retry with backoff (bounded), then degrade to "skip this source this run" and log it — never halt the whole ingest batch for one dead source |
| Auth expired | Saved session/cookie no longer valid | Halt that source's ingestion, flag for re-authentication; do not attempt to guess or re-derive credentials |
| Schema violation | Agent output fails validator | Retry per the contract-enforcement budget above, then halt and escalate to the human |
| Empty result | Retrieval query returns nothing above confidence threshold | Degrade: surface "no supporting source found" to synthesis rather than letting it draft an unsourced claim |
| Partial result | PDF table extraction gets 8 of 10 rows | Degrade with an explicit gap marker in the normalized record — never silently accept partial data as complete |

## 7. The five pillars

### 7.1 Ingest

**Requirements.** Scheduled and on-demand acquisition from three source
types: web sources (blogs, research pages, press releases), document drops
(fund PDFs, PPMs, LP reports, consultant papers, 10-Ks), and manual
paste-ins. Must handle JavaScript-rendered pages, paywalls she legitimately
subscribes to, and PDFs that are scanned images rather than text. Must be
polite: robots.txt compliance, rate limiting, an identifiable user agent, and
a per-source ToS record kept in the provenance ledger.

**Chosen approach.** Sources are config-driven, not hardcoded per connector —
a source registry entry (URL pattern, fetch strategy, auth requirement,
schedule, robots.txt/ToS status) is what ingest reads, so adding a source is
a config change, not new code, until a source needs genuinely bespoke
handling. `[ASSUMPTION: the concrete initial source list was not supplied —
the process this PRD follows explicitly forbids inventing one. The registry
design above is chosen specifically so that filling it in later is additive,
not a redesign.]`

The local-Mac-plus-GitHub-Actions deployment split (confirmed) creates a real
architectural boundary, not just a scheduling choice: **GitHub Actions runs
ephemeral containers with no access to her authenticated browser session.**
Sources reachable without login (public blog HTML/RSS, press releases,
public regulatory filings) are scheduled as cron-triggered GitHub Actions
workflows that run the ingest skill's CLI directly and push captured raw
files + provenance records to the corpus store. Sources behind a login or
paywall she subscribes to require a real Playwright session with
`storageState` persisted from an authenticated local session — those run
on-demand, locally, initiated from a Claude Code session on her Mac, never
from Actions. `[ASSUMPTION: no source has been classified yet as
authenticated vs. public — the registry entry's auth field is where that
classification lives once the source list exists.]` Licensed vendors
(PitchBook, Bloomberg, Morningstar-class) are excluded from both paths by
default pending §9's ToS review — see that section before building any
connector against one.

Scanned-image PDFs get OCR at ingest time (raw capture) with the OCR
confidence score carried forward as a field on the record, so normalize and
retrieve can treat low-confidence OCR text differently from clean extracted
text rather than trusting both equally.

**Alternatives considered.** A single always-on ingest daemon (local or
cloud) was rejected: it doesn't fit "local Mac, not always running," and it
collapses the authenticated/unauthenticated boundary that GitHub Actions'
ephemeral, session-less environment forces to be explicit — a boundary that's
useful to keep explicit anyway, since it's exactly the line licensed-vendor
ToS review needs to sit on.

### 7.2 Normalize

**Requirements.** Boilerplate/nav-chrome stripping, deduplication across
near-identical syndicated copies, table extraction from PDFs into structured
form, entity resolution (the same manager, fund, or index under six
spellings), unit/currency normalization, and as-of-date stamping. Every
record keeps a pointer to its source and capture timestamp — non-optional.

**Chosen approach.** Normalize is where the schema registry (§6) starts
mattering operationally: a normalized record is the first artifact with a
locked shape that retrieve, synthesize, and publish all depend on, so its
schema is versioned from day one even though the pillar itself starts thin.
Entity resolution begins from `[ASSUMPTION: no seed taxonomy of managers/
funds/indices was supplied — normalize starts with an empty alias table and
grows it as records are processed, using fuzzy match plus an agent judgment
call for ambiguous cases, with her confirming merges above a low-confidence
threshold rather than the system silently merging two entities that happen
to look similar.]` Deduplication runs on normalized text (post
boilerplate-strip), not raw HTML, since two syndicated copies of the same
release rarely match byte-for-byte but do match after chrome removal.

**Alternatives considered.** Running entity resolution as a fully automated,
no-human-in-the-loop process was rejected — a wrong auto-merge (two distinct
funds treated as one) corrupts every downstream retrieval and citation
silently, which is a worse failure mode than asking her to confirm an
ambiguous merge occasionally.

### 7.3 Retrieve

**Requirements.** RAG over the normalized corpus, with metadata filters that
make retrieval usable in this domain: asset class, manager, geography,
document type, as-of date, and source confidence. Answers must cite source
documents with page or paragraph anchors.

**Chosen approach.** Hybrid retrieval (dense + BM25) with reranking is
adopted as the default, per the source process's stated hypothesis — this
domain has both semantic queries ("funds with a similar strategy to X") and
exact-term queries ("show me every mention of a specific fund's Q2 NAV"),
and BM25 covers the latter far better than dense retrieval alone, which tends
to blur exact figures and proper nouns. `[ASSUMPTION: embedding model and
vector store are unresolved — Anthropic does not ship a first-party
embeddings API, so this needs either a partner embedding model (e.g. Voyage
AI, Anthropic's recommended partner) or a local embedding model, and the
choice depends directly on the data-residency answer that's still open in
§9. Not decided here.]` Given the confirmed local-Mac deployment and
`[ASSUMPTION: 6–24 month corpus size of low-hundreds to low-thousands of
documents — not confirmed]`, a single-node local vector store (e.g. a
file-backed store rather than a hosted cluster) is the working assumption
until real corpus-size numbers say otherwise.

Every retrieval result carries its confidence/source-quality metadata through
to synthesis — a hybrid-retrieval hit from a licensed, dated primary document
should be weighted differently by synthesis than one from an unattributed
blog aggregator, and that distinction has to survive the retrieval → synthesis
handoff as data, not be re-derived by synthesis from scratch.

**Alternatives considered.** Dense-only retrieval was considered and rejected
per the reasoning above. A managed/hosted vector database was considered and
deferred rather than rejected outright — it's the right call if corpus size
or query volume outgrows a local store, but adopting one now would mean
sending fund and LP document content to a third-party host before the
data-residency question in §9 is resolved.

### 7.4 Synthesize

**Requirements.** Research memos, IC memos, competitive-intelligence briefs,
and benchmarking analyses, written in the user's voice — prose-forward,
argumentative, institutional-finance register, structurally disciplined
without being bullet-slop. Numeric claims must be traceable and verified;
a claim without a source must be blocked from reaching a draft.

**Chosen approach.** Voice is captured as a combination of a written style
guide (register, sentence structure, banned generic phrasings) plus few-shot
exemplars drawn from her own prior work, with a critic pass that specifically
checks a draft against the style guide and rejects generic-AI phrasing before
a draft is considered ready for her review. `[ASSUMPTION: no exemplar
documents were supplied yet — synthesize ships Phase 1 with a generic
institutional-finance style guide only, and the few-shot layer activates once
she points to a folder of prior memos/posts. This is one of the highest-
leverage inputs still missing: voice quality depends on it directly.]`

The numeric layer is enforced structurally, not by instruction: every figure
that appears in a draft must resolve to a specific retrieval-layer citation
(source document + anchor) carried as data alongside the draft, not just
narrated in the prose. A deterministic check (skill layer, not agent
judgment) runs before a draft is considered complete and rejects any numeric
claim lacking a resolvable citation — this is the concrete mechanism the
source process asks for, not an aspiration to "always cite sources."

`[ASSUMPTION: the three deliverable types that matter most in the first 90
days were not ranked — research memo, competitive-intelligence brief, and one
HTML/Chart.js deck are used as the Phase-1-relevant assumption based on her
stated pain points, but this should be confirmed before Phase 2 scopes
synthesis templates.]`

**Alternatives considered.** Relying on prompt instructions alone
("write in her voice," "always cite your sources") without the critic pass or
the structural citation check was rejected — both are exactly the kind of
soft constraint that degrades under the non-determinism the architecture
already flags at the agent layer (§4); a deterministic gate is required
precisely because the layer producing the draft isn't one.

### 7.5 Publish

**Requirements.** Three output surfaces sharing one design system: (a)
HTML/Chart.js decks and dashboards, (b) PowerPoint via `pptxgenjs`/
`python-pptx` with matplotlib chart assets, (c) newsletters and long-form
Word/Markdown documents.

**Chosen approach.** She has already built HTML/Chart.js decks and a
chartbook PPTX rebuild by hand — these are brownfield patterns to formalize
into the token-driven design system (§8), not greenfield surfaces to invent
from scratch. `[ASSUMPTION: she will point to the specific existing
file(s) — "I will point you to the existing code when the time is right."
Phase 1's thin slice (§10) targets reproducing one of these two existing
artifacts end-to-end through the pipeline; which one is an open decision
until she names it, not a default this document should guess at.]`

Nothing publishes without passing the review gate specified in §9 — publish's
last step is always "produce a reviewable draft," never "ship."

**Alternatives considered.** A single unified templating engine that emits
all three formats from one abstract document model was considered and
deferred as premature: the three surfaces have different native idioms
(a slide deck is not a linearized memo with page breaks), and forcing them
through one abstraction before the token-driven design system (§8) proves
itself on real decks risks the token file being wrong in ways that are
expensive to unwind across three renderers instead of one.

## 8. Design system and token architecture

One token file — color, type scale, spacing, chart palettes, table styles —
compiles to four consumers: CSS custom properties (for the HTML/Chart.js
surface), a Chart.js theme object, a matplotlib style sheet, and a
`pptxgenjs` theme. This is the single source of truth for visual identity
across all three publish surfaces; a change to the brand palette or type
scale is one edit, not three.

Charts must be legible in grayscale print and colorblind-safe. `[ASSUMPTION:
specific accessible palette values are deferred to the `dataviz` skill's
documented default palette (brand-neutral, validated for colorblind safety)
as the Phase 1 starting point, swapped for Northern Trust's approved palette
once brand assets are available — see below.]`

Two constraints, stated now rather than discovered later: Northern Trust's
logo, wordmark, and official brand assets are governed by the firm's brand
and compliance functions. This system must be *coherent with* NT's visual
language and structured so approved assets can be dropped into the token
file, but it must not ship unapproved reproductions of trademarked marks.
`[ASSUMPTION: no NT-approved brand assets were supplied — the token file
ships Phase 1 with a neutral placeholder palette and no logo/wordmark
files, clearly marked as pending brand review.]` Second, the token file
itself is the thing that gets brand review — approval happens once, at the
token level, not per-deliverable.

## 9. Compliance, data licensing, and risk

This is regulated financial services; every item below needs a concrete
answer before the system handles anything sensitive, not just an aspiration.

**Licensed data terms.** PitchBook is her primary private markets source;
Bloomberg, Morningstar, and similar vendors carry terms that typically
restrict scraping, caching, and redistribution. `[ASSUMPTION: none of these
vendors' current ToS have been reviewed as part of this conversation. Default
posture, stated explicitly rather than resolved silently in favor of
ingestion: no licensed vendor is scraped. If she has a sanctioned export or
API entitlement for a given vendor, that vendor moves from "off-limits" to
"ingested via that sanctioned path" — but that is a per-vendor decision that
needs her confirmation, not an assumption this system makes on its own.]`
This is flagged as ambiguous on purpose, per the source process's explicit
instruction not to resolve licensing ambiguity silently.

**MNPI and information barriers.** Fund documents and LP reports may contain
material non-public information. Two separate concerns here, and they need
separate answers:

1. *Segregation within the corpus.* Records derived from documents likely to
   carry MNPI (LP reports, PPMs, non-public fund materials) should carry an
   explicit sensitivity tag at normalize time, and retrieval should support
   filtering them out of any query context that doesn't need them — not as a
   hard technical barrier (this is a single-user system today, per §2), but
   as the tagging infrastructure a real access boundary would need if a
   second seat is ever added.
2. **Data leaving the local environment at all.** This is the compliance
   flag this document is not willing to wave off: **Claude Code sending
   content to Anthropic's API means MNPI-tagged content, if it ever reaches
   an agent step, leaves her machine and reaches a third party.** Confirming
   "use Claude, don't worry about the rest" answers *which model*, not
   *whether sending confidential fund content to any external API is
   acceptable under the firm's data-handling policy.* `[ASSUMPTION: not
   resolved. This needs an explicit answer — from her, or from whoever at
   the firm owns data-handling policy — before any MNPI-tagged document is
   allowed to reach an agent-layer step. Until answered, the conservative
   default is: MNPI-tagged records are ingested and normalized, but excluded
   from any agent invocation (synthesis, retrieval-query agents) by a
   deterministic filter at the skill boundary, not by agent-layer judgment.]`

**Attribution and hallucination control.** Enforced by the mechanism in
§7.4: every numeric claim in a draft must resolve to a stored citation
(source document + anchor) checked by a deterministic skill-layer validator
before a draft is marked complete — not by asking the synthesis agent to
"remember to cite sources."

**Review gates.** Nothing publishes without human review. The gate sits at
the end of the synthesize stage, before publish renders a final-format
artifact: the reviewer (her, per §2, until a formal compliance role is
added) sees the drafted content alongside its full citation trail — every
numeric claim next to the source document and anchor it resolved to — so
review is "check the sourcing," not "re-derive it from scratch."

**Retention and audit.** `[ASSUMPTION: no retention policy was specified.
Working default: raw captures, normalized records, and the provenance ledger
are retained indefinitely on local storage (and in the GitHub Actions-fed
corpus store) since storage cost at the assumed corpus size (§7.3) is
negligible, with the provenance ledger itself serving as the audit trail —
every record answers "what source, captured when, under what ToS status,
used in which published deliverable." This should be revisited once a real
firm retention policy is confirmed, since financial services retention
requirements are often a floor, not a ceiling, and may mandate specific
retention periods this default doesn't yet account for.]`

## 10. Milestones

**Phase 1 — Module 0 plus one working end-to-end thin slice.** Not a
finished ingest layer with nothing downstream. Concretely: (1) the plugin/
namespace verification spike (§5) confirming `<plugin>:<skill>` resolution
on the installed Claude Code version; (2) the thinnest version of each
Module 0 piece (§6) that a single real pipeline run needs — one schema, one
validator, minimal observability trace, the error taxonomy's halt/retry
behavior wired for at least the schema-violation and source-unreachable
classes; (3) one source ingested end-to-end through normalize, retrieve, and
synthesize into a draft resembling one of her two existing brownfield
artifacts (§7.5) — which one is `[ASSUMPTION: pending — she will name the
target artifact]`. Phase 1 is done when that one artifact's draft reaches the
review gate with a working citation trail, not when the builders are elegant.

**Phase 2 — Widen ingest and formalize synthesis templates.** Add the
confirmed initial source list (§7.1, pending) across both the local and
GitHub Actions ingest paths; resolve the embedding/vector-store choice (§7.3)
against real early corpus size; add the few-shot voice layer to synthesis
once exemplars are available (§7.4); confirm and build out the top three
90-day deliverable types (§7.4) as named synthesis templates rather than one
proof-of-concept.

**Phase 3 — Full design-system rollout and licensed-vendor resolution.**
Bring in NT-approved brand assets once brand review clears them (§8); resolve
the per-vendor licensed-data posture for PitchBook/Bloomberg/Morningstar-class
sources (§9) and build sanctioned-path connectors for whichever are cleared;
extend publish templates to cover all three surfaces for all confirmed
deliverable types; revisit the MNPI/data-residency decision (§9) if it
changed the architecture materially in Phase 1–2.

## 11. Success metrics

- **Phase 1 gate:** one real source's content travels ingest → normalize →
  retrieve → synthesize → a review-ready draft of a named existing artifact,
  with every numeric claim in that draft resolving to a stored citation —
  observed by inspecting the run's provenance trace, not asserted.
- **Reuse, not re-derivation:** a correction made to a normalized record (a
  fixed figure, a corrected entity name) is observably reflected in every
  subsequent draft that queries it, without a separate manual fix per output
  surface.
- **Citation coverage:** 100% of numeric claims in a published deliverable
  resolve to a source document and anchor — measured by the deterministic
  validator in §7.4/§9, not by spot-check.
- **Zero silent licensing violations:** zero ingested records originate from
  a licensed vendor's site via scraping where that vendor's ToS was not
  explicitly reviewed and cleared (§9) — the default-deny posture is
  observable as "no connector exists" for anything not on the cleared list.
- **Time-to-first-draft:** `[ASSUMPTION: no baseline was supplied to compare
  against. Once Phase 1 ships, measure wall-clock from "source captured" to
  "review-ready draft" for the target artifact, and treat that as the
  baseline this system needs to beat on the next comparable deliverable —
  rather than inventing a percentage-improvement target now.]`
- **Plugin architecture holds:** adding the second and third pillar plugins
  after Phase 1 requires zero changes to the meta-layer's builders or schema
  registry — confirms the "add a plugin, not deepen a tree" scaling claim in
  §5 rather than assuming it.

## 12. Open questions and decisions deferred

All flagged inline above, collected here for a single pass of confirmation:

1. **MNPI / data residency** (§9) — can any MNPI-tagged content reach an
   agent-layer step (and therefore Anthropic's API) at all, under the firm's
   actual data-handling policy? This is the single highest-priority open
   question in this document.
2. **Licensed vendor posture** (§9) — per-vendor confirmation for PitchBook,
   Bloomberg, Morningstar-class sources: sanctioned export/API access
   available, or fully off-limits?
3. **Initial source list** (§7.1) — the 10–20 named sources, not invented
   here by design.
4. **First deliverable to automate** (§7.5, §10) — which specific existing
   HTML/Chart.js deck or PPTX chartbook Phase 1 targets, and the file
   path(s) to it.
5. **Embedding model / vector store** (§7.3) — blocked on the data-residency
   answer in #1.
6. **Corpus size at 6/24 months** (§7.3) — even an order-of-magnitude
   estimate changes the vector-store sizing assumption.
7. **Voice exemplars** (§7.4) — a folder or set of prior memos/posts to seed
   the few-shot layer.
8. **Top three 90-day deliverable types**, ranked (§7.4) — confirm or correct
   the memo/competitive-intel-brief/deck assumption.
9. **Entity taxonomy seed** (§7.2) — confirm there is no existing
   manager/fund/index list to start from.
10. **NT brand assets** (§8) — availability and timing of brand review.
11. **Retention policy** (§9) — confirmed retention period, if the firm has
    one that's stricter than "keep everything."
12. **Retry budget for agent-layer validation failures** (§6) — the
    2-retries/3-tries figure is a placeholder, not a confirmed number.
