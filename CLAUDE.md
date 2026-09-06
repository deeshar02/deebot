# CLAUDE.md

> **Template usage:** This file is what every agent session reads first. Fill in
> the placeholders when starting a project — especially the **Commands** table,
> which `/dev:plan`, `/dev:build`, and `/dev:handover` rely on for validation.
> Until it's filled in, those commands say so and skip the checks rather than
> guessing at a stack.

## Project

{One paragraph: what this project is and its current phase. Link to `PRD.md` for
scope and `PROGRESS.md` for status.}

## Commands

Every check the workflow runs is discovered from this table — no `/dev:*` command
assumes a stack. Delete any row this project genuinely doesn't have (plenty of
projects have no separate type-check step); leave a row as `{placeholder}` only
while the project is still being set up.

| Purpose | Command |
|---|---|
| Dev server / run | `{npm run dev}` |
| Build | `{npm run build}` |
| Lint | `{npm run lint}` |
| Type-check | `{npx tsc --noEmit}` |
| Tests | `{npm run test}` |
| Deploy | `{command or "via CI on merge"}` |

## Conventions

- {Component/module organization — where things live, what to reuse before creating new}
- {Styling approach and its rules}
- {Data access patterns — where clients/integrations live}
- {Forms/validation approach}
- {Anything the linter can't catch: naming, error handling, commit style}

## Workflow

This project uses the PRD → plan → build → handover loop:

1. `/dev:onboard` — orient a fresh session (`/dev:onboard-learning` for a plain-language version)
2. `/dev:plan <module or task>` — write an executable plan to `docs/plans/`
3. `/dev:build docs/plans/<file>.md` — execute the plan with per-task validation
4. `/dev:handover` — run the checks, update `PROGRESS.md` / plan / memory, hand off cleanly

## Workflow rules

These hold for every `/dev:*` command. Each command file states only the rules
particular to itself and leans on these for the rest.

1. **`PRD.md` is the contract; `PROGRESS.md` is the status.** Scope questions are
   answered by the PRD. What's done is answered by `PROGRESS.md` — not by reading
   the code and inferring.
2. **Planning and building are separate passes.** `/dev:plan` writes a plan and
   changes nothing else. `/dev:build` executes a plan and invents nothing else.
   Don't collapse the two because a task looks small.
3. **Nothing is marked done that hasn't passed a check that actually ran.** Not
   "should pass", not "the change is obviously correct". Either the check ran and
   exited clean, or the item stays open.
4. **Paths in a plan are authoritative.** If a path doesn't exist, or contradicts
   what's on disk, stop and reconcile. Never quietly create the file somewhere else.
5. **Take commands from the table above.** If a needed row is still a
   `{placeholder}`, say so and skip that check. Never substitute a guess.
6. **Work outside the plan is reported, not silently done.** Note it and let the
   user decide — unasked-for changes are the hardest kind to review.
7. **Memory holds durable facts only** — preferences, constraints, non-obvious
   gotchas, and a pointer to the active plan. Branch names, commit hashes and file
   lists live in git, so they never belong in memory.

## Gotchas

- {Non-obvious things a new session must know — env quirks, flaky checks, load-bearing hacks}
