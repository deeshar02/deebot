```
---
description: Write a single-pass executable plan for a module or sub-task
argument-hint: [module-or-task-description]
---

# Plan

Produce an executable plan for: `$ARGUMENTS`

## Rules

Beyond the **Workflow rules** in `CLAUDE.md`:

- **Write the plan, not the code.** The only file this command creates is
  `docs/plans/{sequence}.{slug}.md`. No source edits, however small or obvious —
  that's `/dev:build`'s pass.
- **Read before you write.** Open the files the plan will touch, so tasks name real
  paths and real symbols. A plan built from guesses fails at the first task, and
  fails in the pass where changes are already landing.
- **Every task carries its own check.** A task with no verifiable check isn't a
  task, it's a hope. See *Validation* below for what counts.
- **Write for a stranger.** The agent that executes this plan won't have seen this
  conversation. Anything you know but don't write down is lost.
- **Split rather than sprawl.** Roughly ten tasks is the ceiling for one plan. Past
  that, it's 🔴 — propose a split instead of writing one mega-plan.
- **Unknowns are named, not smoothed over.** If something can't be settled without
  running code or asking the user, write it into the plan as an open question with
  the decision it blocks. Don't paper over it with a plausible-sounding task.

## Process

1. **Locate context**
   - `PRD.md` — the relevant module's *Build* / *Delivers*
   - `PROGRESS.md` — what's already done, and what blocks the requested work
   - `docs/plans/` — any earlier plan that overlaps, so this one extends it rather
     than contradicting it
   - The source files the plan will touch

2. **Decide fit and complexity**

   State what this reuses and what it must not duplicate — existing components,
   utilities, config, patterns. Then add a complexity indicator at the top:

   - ✅ **Simple** — single-pass executable, low risk
   - ⚠️ **Medium** — may need iteration, some unknowns
   - 🔴 **Complex** — break into sub-plans before executing

   If 🔴, stop and propose the split. Don't write the mega-plan first.

3. **Write the plan to** `docs/plans/{sequence}.{slug}.md`

   Sequence matches the PRD module number where possible (`2.data-layer.md`,
   `3.1.auth-flow.md`). Required sections:

   - **Goal** — one sentence, tied to the PRD module
   - **Fit** — where this slots in, what it reuses, what it must not duplicate
   - **Pre-conditions** — branch, env, deps, credentials, anything that must
     already be true before task 1
   - **Tasks** — ordered; each names the file paths to touch, the change, and at
     least one validation step
   - **Validation** — the overall success check: what is observably true when the
     whole plan has landed
   - **Rollback** — how to revert cleanly if it fails partway. Be specific: which
     commit to revert, which generated files or migrations to undo, which config
     to restore. `/dev:build` reaches for this section when a task fails twice, so
     "git revert" alone is not enough.
   - **Open questions** — anything unresolved, and what it blocks. Omit the section
     only if there genuinely are none.

4. **Validation is mandatory**

   Every task needs at least one verifiable check, using the Commands table in
   `CLAUDE.md`. What counts:

   - A build / lint / test command exits 0
   - A `curl` or HTTP request against a running instance returns expected content
   - A DOM or API assertion in the project's test framework
   - A manual step with an exact, observable expected outcome — "the page loads and
     shows N rows", not "verify it looks right"

   What doesn't count: "check the code compiles mentally", "confirm the change is
   correct", or any check whose command is still a `{placeholder}` in `CLAUDE.md`.
   If the project has no runnable check for a task, say so in the plan and give the
   most specific manual step you can.

## Output

- Path to the plan file
- Complexity indicator
- One-paragraph summary of the approach, and how it fits what's already there
- Any open questions the user needs to answer before building
- Hand off: "Next: `/dev:build docs/plans/{file}.md`"
- ⁠
```