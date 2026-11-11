# Fixer-report, code review round 2

## Changes

- F1, fixed as proposed: `internal/gatecli/doc.go` now says an entry with no recorded measurement time counts as the oldest.
  Commit `gatecli: fix F1`.
- F2, fixed as proposed: `TheAuditRunsOnlyWithNothingQueuedAndASlotFreeAndCountsOnlyADifferentResult` gained a step before the equal-result call.
  `TestCallerAdded` has a zero `MeasuredAt` and cleared blocks, `TestCallerCalls` is dated 2000, and the audit must take `TestCallerAdded` first, count one mismatch and set its blocks and time again.
- F3, fixed with a wider change than proposed: the review proposed two rows (a test hang, a coverage build hang).
  The premise names three build-side cancel checks (after `LinkedPackages`, after `BuildBinary`, inside `buildLym`), and the self-check showed two rows left two of them uncovered.
  `ACallEndedDuringABuildOrATestRecordsNoEntryAndNoFailure` is now a table of four rows: `go list`, coverage build, `lym` build and test.
  The stand-in gained two hang switches, `testMapHangListEnv` and `testMapHangLymBuildEnv`.
  The rows run with `prebuilt_lym` on the mapped tiers and a committed `cmd/lym`, restoring the gate file on cleanup.
  Each row asserts the cancelled call's report and that `ReadMap` finds no map, so neither an entry nor a `BuildFailure`/`Failure` was written.
- F2 and F3 share one test file and went into two commits (`F2, F3` for the first rows and `F3` for the extra rows).
- Gate: `lym gate test ./internal/gatecli` tier 0 and tier 1 pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, the F3 coverage-build row covers one of three build-side cancel checks. Fixed with my own change (the extra rows above).
- fix-affected-set: MEDIUM, same F3 gap, merged with the fix-proposal finding. Fixed with the same change.

## Disputed

None.

## Deferred

None.

## Rulings

None.
