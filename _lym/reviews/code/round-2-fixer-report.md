# Fixer report — code round 2

## Changes

- **F1** (fixed as proposed): `audit` in `internal/gatecli/testmapaudit.go` calls `r.readHead()` at the top of each package iteration, after the context check and before the `headMoved` test, and returns its error. Its doc comment says so.
  The coverage row the fix asked for is deferred; see below.
- **F2** (fixed as proposed): `registerUpkeepPass` in `internal/gatecli/testmap.go` holds a write lock, `testmap-upkeep.lock` in the hub's scratch directory, across the `runproc.List` and `runproc.Register` pair.
  The refusal message and return values are unchanged.
- **F3** (fixed as proposed): `runRounds` appends to `r.built` only when the package is not yet in it, and the deferred append in `mapPackage` goes through the new `addMeasure`, which sums a resumed job's build seconds, run seconds and tests into the package's existing measure.
  `TestGateTestMapPassYields` gained the row `ResumedPackageIsBuiltAndMeasuredOnce` (no hand call), asserting one built entry, one measure, the second job's timed-out hanging test and at least two measured tests.
  The pass's `secondUnit` in that test is now 500ms so the resumed hang times out in 10 seconds.

Gate: `lym gate test ./internal/gatecli` (tier0) and `--tier 1` passed before the comment repair below.
Commits: one per finding, plus one follow-up for the self-check's test findings, plus one for the gate repair.

## Gate repair

The gate found a wrapped comment break on the doc comment of `testMapRegistrationLockName` in `internal/gatecli/testmap.go`.
The comment is now two sentences, one per line, with no mid-sentence break.

## Self-check

- fix-code-correctness, LOW: the new row would pass if the resumed package ran one job only. Fixed as proposed: asserts `TimedOut` and the measure's `Tests` of at least two.
- fix-code-correctness, LOW: no coverage row for F1. Deferred; see below.
- fix-code-correctness and fix-proposal (merged), LOW: the early `return` in the `!handCall` branch skipped the exit-record assertion. Fixed as proposed: the branch no longer returns, and the `afterPass` comparison alone is conditional.
- fix-consistency, LOW: the `TestGateTestMapPassYields` doc comment named two rows of three. Fixed as proposed.
- fix-consistency, LOW: the 20 second comment is 10 seconds after the `secondUnit` change. Fixed as proposed.
- fix-consistency, LOW: `testmap-upkeep.lock` is not listed in `internal/fabricengine/doc.go`. Disputed, `premise false`: the doc lists the `runproc` record directory (`internal/fabricengine/doc.go:1198`) and is not a list of every scratch-directory lock.
- fix-proposal: no other findings.
- fix-affected-set: no findings.

## Disputed

- Self-check finding on `internal/fabricengine/doc.go`, ground `premise false`: its lines naming `<HubScratchDir>/runproc` describe the push children's record directory, and a grep for `runproc` and `hub-workspace.lock` there shows no exhaustive scratch-directory lock list to keep current.

## Deferred

- F1's coverage row (HEAD moves after the last queued job and before the audit, asserting no audit job builds): the move has to land between two jobs of one call, and the stand-in harness can only move HEAD from inside a test run, which `endRound` already ends the job for.
  No deterministic seam exists in the harness; adding one is a change to `runTestMap`'s test hooks outside this finding.

## Rulings

None.
