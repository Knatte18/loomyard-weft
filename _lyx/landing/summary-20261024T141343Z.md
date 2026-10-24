# gate test runs every tier and guard run past a failing one

`lyx gate test` no longer stops at the first failing tier.
A failure the change under test did not cause, such as one already present at the base, used to hide every later tier and the guard step, so a batch's tier1 compile, its vet steps and its guard step went unchecked with no sign they were skipped.
Now every listed tier runs, then every guard run of the plan, whatever the earlier outcomes.

Closes #864

## Behavior

- A run that started its command and exited non-zero counts as failed, a timed-out tier and an empty-tier failure included, and stops no later run.
- The exit code is the first failing run's own, in plan order, so a call where one tier fails exits as before.
- When any run failed, the verb ends with one stderr line naming each failed run with its exit code, a guard run as `guard tierN`.
  A call where every run passes prints none.
- A refusal (stdout stays one JSON envelope), a slot wait that ends without the slots held, and a cancelled context from a terminating signal still end the call at once.
- A halt is never green: a halt after an earlier failure exits with that failure's code and prints the closing line, noting that later runs did not run.
  A halting run that ended without a failure exits 1 unless a signal was caught.
  A busy hub (75) after a failed run exits with the failure's code, since 75 reads as no failure.
- The green record still records only passes, so a failed tier never carries.

## Design decisions

- The issue offers two fixes, running past the failure or listing what was not reached.
  Running past covers both asks, since every step is then reached, so no "not reached" listing exists.
- The loop in `internal/gatecli/run.go` walks the plan once, in order; guard runs come last in the plan, so `guardRunOf` is gone.
  Each run returns a `runEnd` (exit code, whether it halts the call, whether its command ran), which keeps a test failure apart from a reason to stop.
- `gateslot.FailedTier` now returns the first failing tier summary line instead of the last, so the tier the landing failure record names matches the exit code.
  The wording follows in `internal/landingshed` (`doc.go`, `verifygate.go`).
- The cost is time only when a run fails: at most each later tier's own `timeout_min`, which a passing call already spends.
- Docs follow in `internal/gatecli/doc.go` (verb, guard step and exit codes), `internal/gateslot/doc.go`, `docs/overview.md`, and the `test` Long text and `--guards` usage in `internal/gatecli/cli.go`; the Long text got shorter, so the CLI-form byte cap holds.

## Risk

A caller reading the gate's output for the first failing tier's lines now also meets later tiers' output.
The per-tier summary lines and run records are unchanged, so a reader keyed on them is unaffected.

## Checks by hand

I did not run the gate tests while writing this description; confirm with `lyx gate test --upto 2` on `main` before pushing.
Worth a manual look, each a finding that was raised and addressed in the diff:

- `internal/gateslot/runlog.go` `FailedTier`: with two failing tier lines in a log it names the first, matching the exit code.
- `internal/gatecli/run.go` loop: a run that exits 0 while the context is cancelled with no caught signal exits non-zero, not green.
- `internal/gatecli/run.go` loop: a refusal, `guardStepRun` error or busy hub after a failed tier keeps the first failure's code and prints the closing line.
- `internal/gatecli/test_integration_test.go`, the guard-table row for a failing tier: a failing guard run is labelled `guard tierN` in the closing line, and the exit stays the listed tier's code.
