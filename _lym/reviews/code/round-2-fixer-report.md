# Round 2 fixer-report

## Changes

- F1 (fixed as proposed, `internal/gatecli/redundant_integration_test.go`):
  `TestGateRedundant`'s main subtest now calls `runRedundant` with a `redundantRequest`.
  It carries `secondUnit: redundantTestSecond` (400 ms), and the production `defaultGoBinary`, `os.Environ()`, `time.Minute` and `defaultPipeWaitDelay`.
  The seam cwd is passed through `lymcwd.WithCwd`.
  The fixture's `redundancy_test_timeout_sec` is 20, so the mutant floor is 4 s and the test bound is 8 s, and the endless `Tally` mutant ends at the floor.
  The comment above the gate string names both bounds.
  `redundantCmd` still passes `time.Second`.
  The subtest took about 41 s before and about 17 s after.
- F2 (fixed as proposed in part, `internal/testcover/sourcescan_test.go`):
  the 16-call loop is a table-driven test with one row per entry of the cycle (outer alone, inner alone) and one row for the tests with no cycle reaching the kit.
  Fixed differently for the reviewer's third row, which holds both cycle tests: it is a row of 16 independent kit-reaching cycles, each with a test at either side.
  One cycle's walk order is random, so a single both-tests row would catch the memo regression about half the time.
  The 16 cycles keep detection at 1 − 2⁻¹⁶ and the correct code always passes.
  The single-entry rows pin each entry's result, but cannot catch the memo regression alone, because it shows only when both entries share one package.

- Gate repair: three comments the gate flagged for a wrapped break or a line of 120 characters or more were rewritten as one sentence per line.
  A second gate pass flagged two lines that still broke at a comma, so those sentences were split in two.
  They are the comment above `TestGateRedundant`'s gate string, the comment above `scanSourceHeader` and the comment of the many-cycles row.
  No code changed.

## Self-check

- fix-code-correctness, MEDIUM: the single-entry rows pass with the memo defect in place. Fixed with my own change: the many-cycles row above.
- fix-code-correctness, LOW: nothing asserts the floor binds. Disputed, every fix worse (see below).
- fix-consistency, LOW: the comment repeated the literal 20. Fixed as proposed.
- fix-proposal, LOW: a 2.5 s floor is thin on a loaded machine. Fixed: the unit is 400 ms, so the floor is 4 s.
- fix-proposal, LOW: the both-tests row was dropped. Fixed with the many-cycles row.
- fix-proposal, LOW: the test no longer pins `redundantCmd`'s `secondUnit`. Disputed, every fix worse (see below).
- fix-affected-set: no findings.

## Disputed

- every fix worse, fix-code-correctness LOW (the floor is not asserted): the kill counts are the same whether a run ends at the floor or the test bound, so the only assertion left is wall time.
  A wall-clock assertion breaks the testing rule against real-time dependence.
- every fix worse, fix-proposal LOW (the production `secondUnit` wiring is unpinned): pinning it needs a real-second wait, which `PATTERN-test-speed` forbids and F1 removes.
  `redundantCmd` stays a one-line `time.Second`, and the refusal subtests and `testmap_integration_test.go:678` still run the verb through `Command()`.

## Deferred

None.

## Rulings

None.
