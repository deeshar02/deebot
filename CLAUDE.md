# CLAUDE.md

> **Template usage:** This file is what every agent session reads first. Fill in
> the placeholders when starting a project — especially the **Commands** table,
> which `/dev:plan`, `/dev:build`, and `/dev:handover` rely on for validation.
> Until it's filled in, those commands say so and skip the checks rather than
> guessing at a stack.

## Project

{One paragraph: what this project is and its current phase. Link to
`.dadai/PRD.md` for scope and `.dadai/PROGRESS.md` for status.}

## Where things live

Two homes, split by audience. `.dadai/` is the machinery the workflow runs on;
`docs/` is written for people to read.

| Path | Holds | Who reads it |
|---|---|---|
| `.dadai/PRD.md` | The contract — what we're building and why | Both |
| `.dadai/PROGRESS.md` | Status, decision log, blockers | Both |
| `.dadai/plans/` | Executable plans, `{seq}.{slug}.md` | Agents |
| `.dadai/config/` | Settings, assets, and the secrets template | Both |
| `docs/` | The wiki — guides, reference, recipes | People |

Rules that follow from the split:

- **Plan filenames are `{seq}.{slug}.md`**, where `{seq}` matches the PRD module
  number (`2.data-layer.md`, `3.1.auth-flow.md`). Not date-prefixed — the
  sequence ties a plan to the module it delivers.
- **Secrets never enter git.** `.dadai/config/secrets.env` is gitignored;
  `secrets.env.example` is committed with keys but no values. See
  `.dadai/config/README.md`.
- **`docs/` is not a changelog.** It holds durable knowledge only — how
  something works, how to do a recurring task. What happened and when lives in
  `.dadai/PROGRESS.md` and git history. `docs/README.md` is the index and the
  authority on what goes in which folder; read it before adding a page.

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
2. `/dev:plan <module or task>` — write an executable plan to `.dadai/plans/`
3. `/dev:build .dadai/plans/<file>.md` — execute the plan with per-task validation
4. `/dev:handover` — run the checks, update `.dadai/PROGRESS.md` / plan / docs / memory

## Superpowers skills

The superpowers skills are used where they earn their place, and overridden
where they'd fight this project's rules. Invoke them with the `Skill` tool.

| Skill | Use it |
|---|---|
| `superpowers:verification-before-completion` | Before any claim that work is done — in `/dev:build` and `/dev:handover`. No caveats; this one just applies. |
| `superpowers:systematic-debugging` | In `/dev:build`, the moment a check fails. |
| `superpowers:brainstorming` | In `/dev:plan`, when the ask is vague enough that planning it straight would be guessing. |
| `superpowers:writing-plans` | In `/dev:plan`, as the quality bar for the plan file. |
| `superpowers:test-driven-development` | In `/dev:build`, for code tasks, once a real test command exists in the Commands table. |
| `superpowers:requesting-code-review` | In `/dev:handover`, before opening a PR. |
| `superpowers:finishing-a-development-branch` | Only when I explicitly ask to wrap up a branch. |

**Where this project's rules win.** These skills were written for a different
workflow. Where they disagree with what's below, what's below is the authority —
don't follow a skill off the edge of this project's conventions:

1. **Never auto-chain into git operations.** `executing-plans` and
   `subagent-driven-development` hand off into branch-finishing that merges,
   pushes and deletes branches. That is overridden by *"don't commit or push
   unless asked"*. Finishing a branch happens when I ask for it, never as a
   step something else triggered.
2. **Never remove a worktree.** `finishing-a-development-branch` offers to run
   `git worktree remove`. Claude Code manages the worktrees under
   `.claude/worktrees/` — including, possibly, the one the session is standing
   in. Ask me; don't run it.
3. **Plans are executed by `/dev:build`.** `writing-plans` adds a header telling
   a future agent to execute the plan through its own machinery. Replace that
   with a pointer to `/dev:build` — the two-pass split is not negotiable.
4. **Two failures, not three.** `systematic-debugging` escalates after three
   failed fixes. Ours stops at two and asks me.
5. **TDD applies to code, not markdown.** Until the Commands table has a real
   test row there is nothing to run — say so and carry on, rather than blocking.
6. **Reporting style is set by "Talking to me".** These skills announce
   themselves and ask one question per message. Don't. Answer first, stay short,
   use the decision format.
7. **`subagent-driven-development` is not the default.** It replaces the build
   loop wholesale. Use it only if I ask for it by name.

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

1. **`.dadai/PRD.md` is the contract; `.dadai/PROGRESS.md` is the status.** Scope
   questions are answered by the PRD. What's done is answered by `PROGRESS.md` —
   not by reading the code and inferring.
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
8. **Docs capture knowledge, not events.** `/dev:plan` and `/dev:build` keep
   `docs/` current, but only where durable knowledge came out of the work —
   how something works, how to do a recurring task. Update an existing page
   before writing a new one, and add every new page to the index in
   `docs/README.md` in the same commit. A build that produced no reusable
   knowledge writes no doc, and says so in a line.

## Gotchas

- {Non-obvious things a new session must know — env quirks, flaky checks, load-bearing hacks}
