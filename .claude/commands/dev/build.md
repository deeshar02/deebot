---
description: Build what a plan describes, checking each step as it goes
argument-hint: [link-to-plan]
---

# Build

Read and execute: `$ARGUMENTS`

## Rules

Beyond the **Workflow rules**, **Superpowers skills** and **Talking to me**
sections in `CLAUDE.md`:

- **The plan is the scope.** Execute what it says. If you spot something worth
  doing that the plan doesn't cover, note it for the report and move on — don't
  fold it in.
- **Validate after every task, one at a time.** Never batch several tasks and check
  once at the end; when it breaks you won't know which one broke it.
- **Two failures on the same task means stop.** Fix once, retry. If the same task
  fails its check again, stop and come to me: what's stuck, in plain English, what
  you tried, and what my options are — including the plan's **Rollback** path and
  what rolling back would undo. Then wait. This overrides
  `superpowers:systematic-debugging`, which would keep going to three.
- **Never chain into git operations.** Finishing a branch, merging, pushing and
  deleting branches happen when I ask, never as a step triggered by finishing a
  plan. See override 1 in `CLAUDE.md`.
- **Reuse before you introduce.** Existing components, utilities and patterns first;
  a new primitive needs a reason you can state in the report.
- **A missing path is a stop, not a decision.** If a file the plan names doesn't
  exist, reconcile with me rather than creating it somewhere plausible.
- **`.dadai/PROGRESS.md` follows reality.** Flip an item only after its check has
  actually passed. Update as you go, not in one sweep at the end — a session that
  gets cut short should leave an accurate file behind.
- **Work silently, report at the end.** No running commentary. One line if you
  change direction mid-way or something takes a while, then the report at the end.

## Process

1. **Read the entire plan first** — every task, its dependencies, its validation
   step, the **Docs** section and the rollback path — before touching anything. If
   the plan has an **Open questions** section with anything unanswered that blocks
   a task, raise it now rather than at that task.

2. **Check the pre-conditions** — branch, env, deps, credentials. If one isn't met,
   stop and say which, in plain English. Don't work around it.

3. **Execute tasks in order**, following the conventions in `CLAUDE.md`:
   - Reuse existing components, utilities and patterns rather than adding new ones
   - Follow the stack and idioms declared in `CLAUDE.md` / `.dadai/PRD.md`
   - File paths in the plan are authoritative
   - For code tasks, use `superpowers:test-driven-development` — but only once the
     Commands table has a real test row. While it's still a `{placeholder}` there
     is nothing to run: say so once and carry on, rather than blocking the task.

4. **Run that task's validation step before starting the next one**

   Use the check the plan specifies for the task, taking commands from the table in
   `CLAUDE.md` — lint, build, tests, or the manual check the plan names. Run only
   the checks that apply to what the task changed; a docs-only task doesn't need a
   full test run. If a needed command is still a `{placeholder}`, say so and mark
   the task unvalidated rather than substituting a guess.

   When a check fails, invoke `superpowers:systematic-debugging` — find the cause
   rather than papering over the symptom. Don't suppress a failure to get past it:
   no `any`, no ignore comments, no skipped tests, unless the plan explicitly calls
   for it. Second failure on the same task means stop and ask me.

5. **Update `.dadai/PROGRESS.md`**
   - Flip sub-items `[ ]` → `[x]` as each one's check passes
   - Parent module to `[-]` in progress, or `[x]` only when *every* sub-item is `[x]`
   - `[!]` for anything blocked, with the reason on the line

6. **Update the docs**

   Do this once, after the tasks are done — the plan's **Docs** section says what
   was expected, but what actually happened wins.

   - **Update before you create.** Read `docs/README.md`, then any page covering
     this area. If the knowledge belongs on a page that exists, it goes there.
   - **Create a page only for genuinely new ground**, in the folder
     `docs/README.md` prescribes — `guides/` for doing a task, `reference/` for
     how something works, `recipes/` for a short specific fix.
   - **Add every new page to the index** in `docs/README.md`, in the same commit.
   - **Write nothing if nothing durable came out of this.** Most builds produce no
     reusable knowledge, and that's the normal case. Never write a doc that
     restates what happened — that's `.dadai/PROGRESS.md`'s job, and duplication
     is what makes a wiki rot.

7. **Verify before you claim it's done** — invoke
   `superpowers:verification-before-completion`. Nothing is reported as passing
   unless its check actually ran and you saw it pass.

8. **Report back** — around 15 lines, in this order:

   1. **What works now that didn't before** — the observable result, in plain
      English. Not the list of files.
   2. **Did the checks pass?** — plain words. "Everything passed" or exactly
      what's still failing and what that means for the project. Never round a
      partial pass up to green, and never call something done that wasn't checked.
   3. **Anything I need to decide or do** — decision format from `CLAUDE.md`.
   4. **What you skipped, changed, or noticed and left alone** — one line each,
      with the reason. This is where deviations from the plan go.
   5. **Docs** — one line: which page you updated or added, or "nothing worth
      documenting" and why.
   6. **How far through the plan we are** — done, and what's left.

   Anything technical — file paths, the commands you ran, check output — goes at
   the very bottom under **Details**, or nowhere if the run was clean. I'll ask
   if I want it.

9. **Suggest the next step** — one line: the next plan in the sequence, or
   `/dev:handover` if this is where the session stops.
