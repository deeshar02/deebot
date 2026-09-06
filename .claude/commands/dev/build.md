```
---
description: Execute a plan from docs/plans/
argument-hint: [link-to-plan]
---

# Build

Read and execute: `$ARGUMENTS`

## Rules

Beyond the **Workflow rules** in `CLAUDE.md`:

- **The plan is the scope.** Execute what it says. If you spot something worth
  doing that the plan doesn't cover, note it for the report and move on — don't
  fold it in.
- **Validate after every task, one at a time.** Never batch several tasks and check
  once at the end; when it breaks you won't know which one broke it.
- **Two failures on the same task means stop.** Fix once, retry. If the same task
  fails its check again, stop, report what failed and what you tried, and offer the
  plan's **Rollback** path. A third improvised attempt is how a bad plan becomes a
  broken repo.
- **Reuse before you introduce.** Existing components, utilities and patterns first;
  a new primitive needs a reason you can state in the report.
- **A missing path is a stop, not a decision.** If a file the plan names doesn't
  exist, reconcile with the user rather than creating it somewhere plausible.
- **`PROGRESS.md` follows reality.** Flip an item only after its check has actually
  passed. Update as you go, not in one sweep at the end — a session that gets cut
  short should leave an accurate file behind.

## Process

1. **Read the entire plan first** — every task, its dependencies, its validation
   step, and the rollback path — before touching anything. If the plan has an
   **Open questions** section with anything unanswered that blocks a task, raise it
   now rather than at that task.

2. **Check the pre-conditions** — branch, env, deps, credentials. If one isn't met,
   stop and say which. Don't work around it.

3. **Execute tasks in order**, following the conventions in `CLAUDE.md`:
   - Reuse existing components, utilities and patterns rather than adding new ones
   - Follow the stack and idioms declared in `CLAUDE.md` / `PRD.md`
   - File paths in the plan are authoritative

4. **Run that task's validation step before starting the next one**

   Use the check the plan specifies for the task, taking commands from the table in
   `CLAUDE.md` — lint, build, tests, or the manual check the plan names. Run only
   the checks that apply to what the task changed; a docs-only task doesn't need a
   full test run. If a needed command is still a `{placeholder}`, say so and mark
   the task unvalidated rather than substituting a guess.

   Fix issues before moving on. Don't suppress a failure to get past it — no `any`,
   no ignore comments, no skipped tests — unless the plan explicitly calls for it.

5. **Update `PROGRESS.md`**
   - Flip sub-items `[ ]` → `[x]` as each one's check passes
   - Parent module to `[-]` in progress, or `[x]` only when *every* sub-item is `[x]`
   - `[!]` for anything blocked, with the reason on the line

6. **Report completion**
   - Tasks completed, and any skipped — with the reason
   - Files created / modified
   - Validation results per task, including anything left unvalidated and why
   - Deviations from the plan, and why
   - Anything you noticed but deliberately left alone

7. **Suggest the next step** — the next plan in the sequence, or `/dev:handover` if
   this is where the session stops.
```