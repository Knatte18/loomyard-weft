# Webster: escalate unfixed verify failures to a cold Opus fixer

A plan-level verify failure in Webster used to go only to fixer forks on Merriam's own model (`sonnet[medium]` by the template).
After the last attempt the run halted to the driver and arbiter, which fix no code.
Now a failure the fork does not clear goes to a cold fixer at the recovery role (`opus[high]` by the template) before the run halts.

## What changes

- The verify gate routes each failed evaluation to a fixer and records `fixer: fork` or `fixer: cold` in the verify-gate report.
  A failure goes cold when it is the last one the attempt budget re-prompts, when an earlier failure of the same shuttle run already went cold, or when the previous fixer added no commit above the pre-fix head.
  Otherwise it goes to the fork.
  The findings' way-forward line and the re-prompt line name the routed fixer.
- The `webster.yaml` template's `verify_gate_attempts` drops from 3 to 2: one fork attempt, then one cold attempt.
  A hub that tunes the cap keeps the cold fixer as its last attempt.
- New verb `lyx webster fix-verify` spawns the cold fixer as its own strand (strand role `verify-fixer`, model from the `recovery` key) and blocks until it ends or `recovery_timeout_min` passes, as `recover-batch` does.
  Merriam runs it backgrounded and acts on the completion notification; the verb joins the Master spawn's awaited shell prefixes.
- The verify-fix stencil is split into a shared body (`webster-body-verify-fix`) and two prefixes, `webster-prefix-verify-fix-fork` and `webster-prefix-verify-fix-cold`.
  The cold prefix is the opening stencil of the `verify-fixer` role and carries the parent, edit, PATTERN and friction directives and the reply-file final action.
- The Master template's gate-failure section tells Merriam to run `fix-verify` instead of spawning a fork when the way forward names it, then rewrite its contract files and end its turn so the gate verifies again.
- `doc.go` for `websterengine` and `webstercli`, `docs/overview.md` and the Webster spec are updated.

## Design decisions

- The escalation lives inside the gate, which already owns the attempt count and the fix-commit range, so it can tell the last attempt and a fixer that committed nothing.
  The tackler and arbiter cannot fix code, so escalating after the halt would not help.
- `fix-verify` is re-entrant: a call that finds a recorded non-terminal fixer attaches and waits instead of spawning a second.
  It reports `done` when the fixer wrote its reply file and `dead` with a reason on a turn end without a reply, a gone strand or the timeout.
  Every terminal call stops the strand, and only `done` removes its run dir.
  A non-done status exits non-zero.
  It refuses, naming the way forward, when no verify-gate report exists.
- The verb counts no spawn of its own.
  The number of cold fixers per run is bounded by the gate messages Merriam receives, which the attempt budget caps, and by the Master template's rule to run it only when a gate message names it.
- `State.VerifyFixers` records each spawn (strand guid, session, engine, time, events path and offset, run dir, terminal status); `state.json` gains an optional key, so older states still load.
  Recorded sessions join the run's write evidence beside the batch recovery sessions.
  The run-entry reclaim stops a non-terminal recorded fixer, marking it dead and saving it, and `reset --to start` stops every recorded fixer strand first.
- The cold fixer's reply file sits in Webster's scratch dir, outside `_lyx`, and is the strand's shuttle output file.
  Its commits pass through the same gate commit check, verify, run-exit audit and Webster-Review as the fork's.
- No new `webster.yaml` key: the cold fixer uses `recovery` and `recovery_timeout_min`.
- Known limits:
  a re-prompt the shuttle could not deliver is re-evaluated on a later tick and counts one more failure, so routing may go cold one attempt early; the cap still bounds the run.
  The cold fixer starts with only the report and the plan overview, so a failure whose cause needs the batch history may still beat it.
- Out of scope: the tackler, the arbiter, the driver, the Merge-In fixer and the batch recovery path.

## To check by hand

- Run a real Webster plan whose verify gate fails twice, and confirm the second failure spawns the cold fixer through `lyx webster fix-verify` and its commits reach the next gate evaluation.
  Tests use injected clocks and stoppers; no live tmux run was made.
