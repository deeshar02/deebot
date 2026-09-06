---
description: Wrap up the session — run the checks, update the docs, leave clean notes
argument-hint: [plan-file-path]
---

# Handover

Close out the session: run the checks, fix what they catch, update the tracking
docs, and leave enough behind that a fresh agent can pick up without asking me
anything.

**Plan file:** `$ARGUMENTS` — if not given, use the active plan noted in memory, or
the most recently modified file in `.dadai/plans/`.

## Rules

Beyond the **Workflow rules**, **Superpowers skills** and **Talking to me**
sections in `CLAUDE.md`:

- **Run the checks that apply to what changed**, not a fixed list. A docs-only
  session doesn't need a full build. Take commands from the table in `CLAUDE.md`;
  if a needed row is a `{placeholder}` or the project genuinely has no such step,
  say so and skip it — never substitute a guess.
- **Fix, don't silence.** No `any`, no ignore comments, no skipped or deleted tests
  to get a check green, unless the plan explicitly calls for it. A suppressed error
  handed to the next session is worse than an open one.
- **Report failures you couldn't fix.** A handover that says "all green" when it
  isn't is the single most expensive thing this command can do. If something is
  still failing, it goes in the report in plain English — what's broken, what it
  blocks, whether it's safe to leave — and `.dadai/PROGRESS.md` gets `[!]`.
- **Git holds the state; memory holds the meaning.** The tracking docs in the repo
  are the durable record. Memory is for what git can't tell the next session.
- **Don't commit or push unless asked.** Leave the working tree for me to review.
  If I do ask, the commit or PR message follows the **Commits, branches and PRs**
  rules in `CLAUDE.md` — written for me, not for a code reviewer.
- **Never finish a branch on your own initiative.** Merging, pushing, deleting
  branches and removing worktrees happen only when I ask for them by name. If I
  do ask, `superpowers:finishing-a-development-branch` is the right tool — but
  present its options in the decision format from `CLAUDE.md`, and never run
  `git worktree remove` yourself: those worktrees are Claude Code's, and one of
  them may be the one this session is standing in. Ask me instead.

## Step 1 — Identify working context

1. Read memory to see which plan is active and what was next.
2. Read the plan file and `.dadai/PROGRESS.md` to understand where the work
   actually got to.
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

Invoke `superpowers:verification-before-completion` before concluding this step.
Keep the real numbers for yourself — you need them to know whether you're done.
What reaches me is the plain version: "everything passed", or exactly what's
still red, what you tried, and what it blocks. A partial pass is never reported
as green.

## Step 3 — Update the tracking documents

These live in git and are the durable record of the work.

**3a. The plan file** (`.dadai/plans/<file>.md`)
- Mark completed tasks ✅; leave upcoming tasks unmarked
- Make the next batch clearly identifiable
- Add anything learned that changes a later task

**3b. `.dadai/PROGRESS.md`**
- Flip sub-items `[ ]` → `[x]` where the check passed
- `[-]` for in-progress modules, `[!]` for blocked, with the reason on the line
- A module is `[x]` only when *every* sub-item is `[x]`

**3c. `docs/`** — only if this session produced durable knowledge that `/dev:build`
didn't already record
- Update an existing page before creating a new one
- Every new page gets a line in the index in `docs/README.md`
- Nothing durable? Write nothing, and say so in a line. `docs/` is not a log of
  what happened — that's `.dadai/PROGRESS.md`.

**3d. `.dadai/PRD.md`** — only if scope genuinely changed
- Update the affected module's *Build* / *Delivers* lines
- Keep it surgical. The PRD is a contract, not a journal.

## Step 4 — Update memory

Memory is a directory of one-fact-per-file entries with `MEMORY.md` as a one-line
index — not a session scratchpad, and not a place to paste a status snapshot.

**Record:**
- Durable preferences or feedback I gave, *with the reason* I gave it
- Non-obvious constraints and gotchas found this session — framework quirks, env
  edge cases, a check that's flaky and why
- One pointer to the active plan and the next task, so a fresh session can orient

**Don't record:**
- Current branch, last commit hash, lists of changed files — git already has these,
  and they're wrong within a day
- Anything already written in `.dadai/PROGRESS.md`, the plan, `docs/`, or
  `CLAUDE.md`
- Anything that only mattered inside this conversation

Update an existing entry rather than adding a near-duplicate, and add a one-line
pointer to `MEMORY.md` for each new entry.

## Step 5 — Report

Around 15 lines, in this order:

1. **Where the work got to** — what's finished and what's next, in plain English.
2. **Is everything healthy?** — the checks in plain words. If something's still
   broken: what it is, what it blocks, and whether it's safe to leave until next
   time. Never "all green" when it isn't.
3. **Anything waiting on me** — decisions, in the format from `CLAUDE.md`.
4. **What you updated** — one line naming the tracking docs and any wiki page.
   Not a diff.
5. **Picking this up later** — a sentence or two that a future session, or I,
   can start cold from without re-reading this conversation.
