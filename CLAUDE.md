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

1. `/dev:onboard` — orient a fresh session
2. `/dev:plan <module or task>` — write an executable plan to `docs/plans/`
3. `/dev:build docs/plans/<file>.md` — execute the plan with per-task validation
4. `/dev:handover` — run the checks, update `PROGRESS.md` / plan / memory, hand off cleanly

## Talking to me

I'm not a software engineer. I read and write Python for analysis, but git,
build tooling and CI are not my world — and I have ADHD, so a wall of text is a
wall I don't read. These rules apply to every `/dev:*` command's report and to
ordinary replies in this project.

They govern what I *read*. They don't govern what you *do*: plan files, code,
commit contents and the checks themselves stay as precise and technical as they
need to be. Don't trade correctness for readability — just don't make me read
the technical version.

1. **Answer first.** Open with what happened, or what you need from me. No
   preamble, no restating my question back at me.
2. **Short.** Aim for 15 lines. If there's more, it goes in a file I can open
   or a "Details" section at the bottom — not in the opening.
3. **Plain English.** "The checks passed", not "lint/tsc/vitest exit 0". If a
   technical term is genuinely unavoidable, gloss it in brackets the first time
   — see the glossary below.
4. **Never fake green.** If something is broken, unfinished or unchecked, say so
   plainly and say what it means for me. A tidy summary hiding a failure is the
   most expensive thing you can hand me.
5. **Say what you assumed.** If you made a judgment call I might disagree with,
   one line: what you picked and what the alternative was.

### When I need to decide something

Don't hand me a technical question and ask me to choose. Give me this, in this
order, then stop and wait:

- **The decision, in one line** — what is actually being chosen.
- **Why it's mine to make** — what you can't settle on your own.
- **The options** — two or three, never five. For each: what it means in
  practice, what it costs me (time, money, risk, being stuck with it), and what
  it gets me. Two lines each.
- **What you'd pick**, and the one-line reason.
- **What happens if I don't decide now** — does everything stop, or can you
  keep going and come back to it?

If a choice is genuinely reversible and low-stakes, it isn't a decision — pick
one, tell me in a line what you picked, and carry on.

### Commits, branches and PRs

When a command creates any of these, what I read is written for me, not for a
code reviewer:

- **Title** — what changed, in plain words. Not file names.
- **Body** — why it changed, and what's different now. If I need to look at
  something or make a call, that goes first.
- **Never** paste a diff, a stack trace, or a list of function names into
  something meant for me. That belongs in the code itself.

When you push or open a PR, add one line on what it means right now: what state
the work is in, and whether it's waiting on me.

### Glossary

Gloss a term the first time it comes up, then use it freely.

| Term | What it means |
|---|---|
| Branch / worktree | A separate copy of the project, so work in progress can't break the version that works |
| Commit | A save point, with a note on what changed |
| Push | Uploading save points to GitHub, so they're backed up and shareable |
| PR (pull request) | A request to fold a branch's work into the main version — the review step before it becomes official |
| Merge | Folding that work in; it's now part of the main version |
| Lint | An automatic checker for style slips and likely mistakes |
| Type-check | A check that data passed between parts of the code is the shape each part expects |
| Tests | Code that runs the project and confirms it behaves as intended |
| Build | Turning the source code into the runnable or publishable version |
| Placeholder | A `{like this}` gap in a template nobody has filled in yet |

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
