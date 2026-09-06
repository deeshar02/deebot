```
---
description: Run all checks, fix issues, update docs and memory for session handover
argument-hint: [plan-file-path]
---

# Handover

Close out the session: run the checks, fix what they catch, update the tracking
docs, and leave enough behind that a fresh agent can pick up without asking you
anything.

**Plan file:** `$ARGUMENTS` — if not given, use the active plan noted in memory, or
the most recently modified file in `docs/plans/`.

## Rules

Beyond the **Workflow rules** in `CLAUDE.md`:

- **Run the checks that apply to what changed**, not a fixed list. A docs-only
  session doesn't need a full build. Take commands from the table in `CLAUDE.md`;
  if a needed row is a `{placeholder}` or the project genuinely has no such step,
  say so and skip it — never substitute a guess.
- **Fix, don't silence.** No `any`, no ignore comments, no skipped or deleted tests
  to get a check green, unless the plan explicitly calls for it. A suppressed error
  handed to the next session is worse than an open one.
- **Report failures you couldn't fix.** A handover that says "all green" when it
  isn't is the single most expensive thing this command can do. If something is
  still failing, it goes in the report and `PROGRESS.md` gets `[!]`.
- **Git holds the state; memory holds the meaning.** The tracking docs in the repo
  are the durable record. Memory is for what git can't tell the next session.
- **Don't commit or push unless asked.** Leave the working tree for the user to
  review.

## Step 1 — Identify working context

1. Read memory to see which plan is active and what was next.
2. Read the plan file and `PROGRESS.md` to understand where the work actually got to.
3. Run `git status` and `git diff --stat` to see what this session really changed —
   that determines which checks in Step 2 are worth running.

## Step 2 — Run the checks

Run from the repo root unless the plan says otherwise. For each one below: run it,
fix what it reports, then re-run it until clean.

- **Lint** — the lint command from `CLAUDE.md`
- **Type-check** — if the project has one
- **Tests** — all must pass; fix failures before continuing
- **Build** — must succeed; note any new warnings even if it passes
- **Anything the plan named** — manual or browser checks specific to this work

Then report real numbers: *N tests passed, 0 type errors, 0 lint errors, build OK* —
or the honest version, naming what's still red and what you tried.

## Step 3 — Update the tracking documents

These live in git and are the durable record of the work.

**3a. The plan file** (`docs/plans/<file>.md`)
- Mark completed tasks ✅; leave upcoming tasks unmarked
- Make the next batch clearly identifiable
- Add anything learned that changes a later task

**3b. `PROGRESS.md`**
- Flip sub-items `[ ]` → `[x]` where the check passed
- `[-]` for in-progress modules, `[!]` for blocked, with the reason on the line
- A module is `[x]` only when *every* sub-item is `[x]`

**3c. `PRD.md`** — only if scope genuinely changed
- Update the affected module's *Build* / *Delivers* lines
- Keep it surgical. The PRD is a contract, not a journal.

## Step 4 — Update memory

Memory is a directory of one-fact-per-file entries with `MEMORY.md` as a one-line
index — not a session scratchpad, and not a place to paste a status snapshot.

**Record:**
- Durable preferences or feedback the user gave, *with the reason* they gave it
- Non-obvious constraints and gotchas found this session — framework quirks, env
  edge cases, a check that's flaky and why
- One pointer to the active plan and the next task, so a fresh session can orient

**Don't record:**
- Current branch, last commit hash, lists of changed files — git already has these,
  and they're wrong within a day
- Anything already written in `PROGRESS.md`, the plan, or `CLAUDE.md`
- Anything that only mattered inside this conversation

Update an existing entry rather than adding a near-duplicate, and add a one-line
pointer to `MEMORY.md` for each new entry.

## Step 5 — Report

- What was checked, what was fixed, and the final results — including anything
  still failing or skipped
- Which tracking docs were updated
- What the next agent should do: active plan, next task, one line of context
- Blockers and open questions
```