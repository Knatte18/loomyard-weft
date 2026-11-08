# Round 1 fixer-report

## Changes

- F1 (LOW), fixed as proposed, commit `1a16f26d`: `TestUnwire_LeavesSiblingWorktreeUndirtied` in `internal/fabricengine/unwire_test.go` gained a `wantStripped` row field.
  After `Unwire` the test reads the exclude file and fails if that anchored line is still present.
  Only `UnwiredPairAloneAtItsAnchor` sets it (`/sub/_lym`); the other rows share their anchored line with the kept pair, which keeps it.
- Self-check follow-up, second commit: a precondition asserts the line is present before `Unwire`, and the test's doc comment names the strip assertion.

- Gate repair: the test's doc comment and the `wantStripped` field comment were rewritten as one sentence per line, with no wrapped break.

The tier 1 gate for `./internal/fabricengine` passes (failed 0).

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Self-check

- fix-code-correctness, LOW, assertion lacked a precondition: fixed as proposed.
- fix-consistency, LOW, test doc comment stale: fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report.
