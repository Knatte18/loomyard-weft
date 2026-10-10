# Fixer report — discussion, round 2

I checked every finding against the code and found each one valid, so all are fixed.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1 (BLOCKING), D2, D4, D11, Acceptance criteria.**
  Confirmed: `proc.Detach` is `Setsid` (`internal/proc/proc_linux.go`), and `ConfigureGroupKill` gives each child its own group on Linux (`procgroup_linux.go`) and kills the child alone on Windows (`procgroup_windows.go`).
  D2 now drops the session premise and ties each child to the loop the way `internal/gatecli` ties `lyx gate test`'s child to its parent:
  - On Linux, `Pdeathsig` SIGKILL, spawned from a goroutine locked to its OS thread.
  - On Windows, a kill-on-close Job Object per child.

  That mechanism moves into `internal/proc` so `internal/shedverbs`' allowlist stays as D4 states it.
  The pid file now also records the in-flight child's pid and start time.
  D4's kill now names the platform kill: the process group on Linux, the Job Object on Windows.
  This also corrects the Windows half of the "grandchild dies too" claim.
  D11's session-end step now does three things in order:
  1. Kills the loop.
  2. Kills the recorded child's group.
  3. Waits a bounded time for the run lock, and fails with the holder's pid if the lock is not released.

  Its bound now also covers a step that started before teardown.
  The teardown acceptance row asserts that the child is dead before the reed session ends.
- **F2 (BLOCKING), D2, D5, D9, Acceptance criteria.**
  The loop writes its pid file only after it takes the loop lock, and removes it only after its envelope is written.
  So a pid file with the loop lock free and no envelope marks a dead loop.
  In that state both a waiter and a later invocation stop as `interrupted` with cause `loop-exited`, and never spawn a fresh loop.
  The stop carries the row's `interrupt_policy` and writes `failed` through D5's function under D5's bound.
  `status_moved` is filled from the child's starting `current_producer` and `history_length`, which the pid file records.
  D5 lists the dead loop among its cases.
  D9's re-run rule now waits on a live loop or reports a dead one.
  Added an acceptance row for a loop killed mid-step.
- **F3 (MEDIUM), D3, Acceptance criteria, Question ledger.**
  Each condition is one slot.
  A second `--before`/`--after` replaces the recorded one, and `--clear` removes both whoever set them.
  In both cases the verb's answer names what it replaced or removed, and the bound is stated.
  Added this to the acceptance row and recorded the choice and its reason in the ledger.
- **F4 (LOW), D3, Constraints.**
  The condition fields are now their own class, outside-set and both-cleared, in D3 and in the `Status` ownership Constraints row.
- **F5 (LOW), D1.**
  The synthesized interrupted form is now the short error form's closed key set and includes `run_id` (`shortStepErrFields`, `internal/shedverbs/step.go`).
- **F6 (LOW), Scope, Acceptance criteria.**
  The Scope line and the acceptance row now limit empty-key dropping to `progress`, matching D7.
  The row states that the step envelope's other keys are unchanged.
- **F7 (LOW), D1, Acceptance criteria.**
  On an interrupted stop the loop now writes a full error record carrying the location keys, and `envelope_path` names that record.
  This makes D7's claim hold for every stop.
  Added an acceptance row.

Per PATTERN-review-round, three residual risks of this round's fixes are carried to `## Open risks`, each marked `(round 2)`:
- The parent-death signal is tied to the thread that spawned the child.
- A loop that dies after its child persisted but before its own envelope is reported as `loop-exited`.
- One actor can override another actor's condition.

## Disputed

None.

## Deferred

None.
