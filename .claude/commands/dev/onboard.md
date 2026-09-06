---
description: Get up to speed on this project and report where things stand
---

# Onboard

Orient a fresh session in this repo and report what's there. This is the first
command of the loop; it changes nothing.

## Rules

Beyond the **Workflow rules** and **Talking to me** sections in `CLAUDE.md`:

- **Read-only.** Never edit, create, or commit anything during an onboard. If
  something looks broken or contradictory, report it — don't repair it.
- **Report the gaps, don't fill them.** A repo freshly copied from the template
  still has `{placeholders}` in `PRD.md` and `CLAUDE.md`. That's expected, not an
  error. Name the files that still need filling in and stop there — never infer
  the project's scope from its file names.
- **Skip checks you can't run honestly.** The health check uses the Commands table
  in `CLAUDE.md`. If the row is still a placeholder, or the command fails because
  dependencies aren't installed, say that instead of trying alternatives.
- **State, not strategy.** Describe what exists and where it stands. Recommending
  what to build next is `/dev:plan`'s job.

## Process

Read widely here — the reading is for you, not for the report. Most of what you
read won't be worth mentioning.

1. **Scan structure**
   - `git ls-files | head -100` to see tracked files
   - List the main source directories to map the app's surfaces (pages/routes,
     components, integrations, services — whatever this project actually uses)
   - `ls .claude/commands .claude/agents` (if present) to see which commands and
     subagents this project has beyond the template's

2. **Read key files**
   - `CLAUDE.md` — conventions, the Commands table, and the workflow rules
   - `PRD.md` — vision, scope, and the module breakdown
   - `PROGRESS.md` — what's done, in flight, and blocked
   - `README.md`
   - The entry point(s) and router/navigation map
   - The package/build manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, …)
     and framework config files
   - The most recent plan in `docs/plans/`, if any — it says what the last session
     was mid-way through

3. **Check state**
   - `git status` and `git log -10 --oneline`
   - Run the lint command from `CLAUDE.md` and skim the tail of the output to gauge
     health. Skip this if the row is still a placeholder.

## Output

Around 15 lines, in this order:

1. **What this project is** — a sentence or two, from `PRD.md`.
2. **Where it stands** — what's finished, what's part-done, what's stuck. Say
   what a blocker actually blocks, in plain English; don't just repeat the `[!]`
   marker from `PROGRESS.md`.
3. **What's waiting on me** — open questions and blockers I need to answer. Just
   the list here. Use the decision format from `CLAUDE.md` only if I pick one to
   go into.
4. **Setup still missing** — any template `{placeholder}` still unfilled, and
   what it stops you doing. "The checks can't run until we say how to run them"
   beats naming the table row.
5. **Anything you couldn't check**, and why.

Leave out the tech-stack inventory, the file tour and the route map unless I ask
— you read them to orient yourself, not to recite them. If something looks
broken or contradictory, flag it in a line; don't fix it.
