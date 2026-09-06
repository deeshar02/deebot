```
---
description: Onboard Claude Code into this codebase
---

# Onboard

Orient a fresh session in this repo and report what's there. This is the first
command of the loop; it changes nothing.

## Rules

Beyond the **Workflow rules** in `CLAUDE.md`:

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

A brief summary:

- **What this project does** and who it's for (from `PRD.md`)
- **Tech stack** — current, and target if a migration is in flight
- **Main surfaces** (routes/pages/services) and which are dynamic or data-driven
- **Where it stands** — current branch, recent activity, uncommitted changes
- **What's in flight** — the `[-]` module in `PROGRESS.md`, anything marked `[!]`
  blocked, and the active plan in `docs/plans/`
- **Still unfilled** — any template placeholder left in `PRD.md`, `PROGRESS.md` or
  `CLAUDE.md`, and any check you skipped and why
```