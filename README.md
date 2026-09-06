# Project Template

A clean starting slate for new projects, built around a PRD-driven agent workflow.
Copy this repo when kicking off a project, fill in the templates with the project's
context, and the `/dev:*` commands handle the plan → build → handover loop.

## What's included

```
.
├── .claude/
│   └── commands/dev/       # Agent workflow commands
│       ├── onboard.md            /dev:onboard — orient a fresh session
│       ├── onboard-learning.md   /dev:onboard-learning — plain-language onboard
│       ├── plan.md               /dev:plan — write an executable plan
│       ├── build.md              /dev:build — execute a plan with validation
│       └── handover.md           /dev:handover — checks, docs, memory handoff
├── docs/
│   └── plans/              # Executable plans land here ({seq}.{slug}.md)
├── PRD.md                  # Product contract: vision, scope, stack, modules
├── PROGRESS.md             # Module checklist ([ ] / [-] / [x] / [!])
├── CLAUDE.md               # Agent context: commands, conventions, workflow rules
└── README.md               # This file — replace with the project's README
```

## Starting a new project

1. **Copy this repo** (or use it as a GitHub template) and rename it.
2. **Fill in `PRD.md`** — vision, users, scope, stack, constraints, and the
   numbered module list. This is the contract everything else hangs off.
3. **Mirror the modules into `PROGRESS.md`** as checklists with sub-tasks.
4. **Fill in `CLAUDE.md`** — especially the Commands table. Every check the
   workflow runs is discovered from it, so until it's filled in the commands will
   report the gap and skip the checks rather than guess at your stack.
5. **Replace this README** with the project's own.

## The workflow loop

```
/dev:onboard                          # orient (once per fresh session)
/dev:plan Module 2                    # → docs/plans/2.{slug}.md
/dev:build docs/plans/2.{slug}.md     # execute, validating every task
/dev:handover                         # checks run, docs updated, memory set
```

Every plan carries its own validation steps and rollback path; `PROGRESS.md` is the
single source of truth for what's done.

## How the commands are written

Each command file has the same three parts, in this order:

- **Rules** — the guardrails, stated before any step. An agent meets them before it
  starts acting, not halfway through.
- **Process** — the numbered steps.
- **Output** — what gets reported back.

The rules shared by all four commands live once in the **Workflow rules** section
of `CLAUDE.md`; each command file adds only the rules particular to itself. When
you change how the workflow behaves, change it there — not in four places.

The load-bearing ones:

- **Planning and building are separate passes.** `/dev:plan` writes a plan and
  touches nothing else; `/dev:build` executes it and invents nothing else.
- **Nothing is marked done without a check that actually ran.** Not "should pass" —
  it ran and exited clean, or the item stays open.
- **A missing path is a stop, not a decision.** If a plan names a file that isn't
  there, the agent stops and asks instead of creating it somewhere plausible.
- **Two failures on the same task means stop** and offer the plan's rollback,
  rather than improvising a third attempt.
- **Memory holds durable facts only** — preferences, constraints, gotchas, and a
  pointer to the active plan. Branch names and commit hashes live in git.
