# Fixer report — code review round 1

## Changes

- **F1** (fixed as proposed, commit `07cac1075`): `gateslot.FailedTier` scans forward and returns the first failing tier summary line; its doc comment, `internal/gateslot/doc.go`, `internal/landingshed/doc.go` and `internal/landingshed/verifygate.go` say "first".
  `internal/verifytree/doc.go` and `pattern/PATTERN-verified-tree.md` never said "last", so they are unchanged.
  A row with two failing lines expecting the first joins the existing `FailedTier` test.
- **F2** (fixed as proposed in effect, commit `2f19920f9`, reworked in the follow-up): a halt can no longer return 0.
  `haltExitCode` returns 1 for a run that ran and halted the call with no failure before it.
  The call prints a stderr line saying why it exits 1.
  Uncovered by a test: no test can cancel the context without a signal.
- **F3** (fixed as proposed, with a refinement): a halt prints the closing line through the shared `printFailedRuns`, which adds "the call stopped there, so later runs did not run".
  A busy hub after a failed run exits with that failure's code; a refusal keeps its own code 1.
  The closing line's wording lost "every later run still ran", which a halt makes false; the test at the failing-tier subtest follows.
  A new subtest, a refusal after a failed tier, pins the line and the exit.
  Its `cmd/lyx` directory is removed on cleanup so it cannot leak into a later subtest.
- **F4** (fixed as proposed): the guard-table row "a failing tier still runs the guards" now fails the guard run too, through `helperGuardExitEnv`, and asserts `tier0 (exit 3), guard tier0 (exit 5)` and exit 3.
- Self-check follow-up (commit after `2f19920f9`): `runEnd` gains `ran`; a run that ran and halted the call joins `failures`, the halt exit is the first failure's code, the green-record pass check reads `end.code == 0 && !end.halt`, and the exit-code text in `internal/gatecli/doc.go`, the `runTest` comment and `docs/overview.md` follows.

## Self-check

- fix-code-correctness, LOW: a halt reporting 0 after an earlier failure exits 1, not the first failure's code. Fixed as proposed, through `haltExitCode`.
- fix-code-correctness, LOW: a halting run that itself failed is missing from the closing line and its code displaces the first failure's. Fixed as proposed, through `runEnd.ran`.
- fix-proposal, MEDIUM: same as the first fix-code-correctness finding. Merged with it.
- fix-proposal, LOW: a halt after a run that passed turns the call red with no message. Fixed: the call prints a line naming the cancellation.
- fix-proposal, LOW: the closing line did not say the call stopped early. Fixed: a halt adds a suffix saying so.
- fix-consistency, MEDIUM: exit-code text in `doc.go`, `run.go` and `docs/overview.md` stale. Fixed in all three. `cli.go`'s sentence "The exit code is the first failing run's own, or 0" stays: it holds for every call that does not halt, and the `test` Long text is byte-capped.
- fix-consistency, LOW: the closing line on a halt undocumented. Covered by the `doc.go` exit-code sentence; the line's wording is pinned by the tests.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
