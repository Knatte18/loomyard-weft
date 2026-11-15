## Fixer report, round 1

Every finding is fixed as proposed, apart from the notes under each entry.
`lym gate test ./internal/impactset` and `lym gate test ./internal/gatecli` passed at tier0.
`lym gate test --tier 1 ./internal/gatecli` passed, which runs the new integration rows.

A later repair fixed the gate's comment findings: two comments of `testmap_integration_test.go` (the stand-in's doc on Overlaps tests and `seesBuildOf`) were re-broken at sentence boundaries under 120 characters.
No finding's fix changed.

## Changes

- **F1** (commit `da2eec27`): fixed as proposed.
  - Deleted `ModuleTargets.Importers` and `BinaryBuilders`, their assignments, `splitUnchanged` and `packageArgsOf`.
  - Deleted the `doc.go` lines that described them.
  - `impactedFor` returned the changed-package set only for `splitUnchanged`, so it now returns `(set, fallback)`.
  - `graph.go` drops its `slices` import.
  - `targets_test.go` asserts `Packages` alone: the `withImporters` helper is gone, and the rows that set `Importers` or `BinaryBuilders` now use `root(...)`.
    Two rows that differed only in those fields keep their different graphs and changed paths.
- **F2** (commit `6262f497`): fixed as proposed.
  - `coverageBuild` has a `cancel` func.
  - `startBuild` runs each build under its own `context.WithCancel(r.ctx)`, passed as the Toolchain's `Ctx`, and the build goroutine defers the cancel.
  - `takeBuild` cancels a build for another package before it waits and removes the binary.
  - `close` cancels the ahead build before it waits.
  - The `startBuild`, `takeBuild` and `close` doc comments say an unused ahead build is cancelled.
  - The matching line of `internal/gatecli/doc.go` ("a build whose package then needs no binary is cancelled and dropped") went into the F4 commit, since both touch that file.
- **F3** (commit `738994c3`): fixed as proposed, with the overlap check written differently.
  - The review proposed timestamped lines. The test uses the order the stand-in records instead, so no clock is compared.
  - A stand-in test ending in `Overlaps` passes only if the coverage-builds file records a build of the package `testMapOverlapEnv` names within `testMapMeetWait`.
  - The existing subtest `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` gained a `lingers` package, whose one test needs the build of the next package, `meets`, while it runs.
    Without the ahead start, the build is recorded only after `lingers` ends, so the test fails and its map entry mismatches.
  - A last subtest of `TestGateTestMap`, `AnAheadBuildTheWalkNoLongerNeedsIsCancelledRatherThanWaitedFor`, runs `amoves` (its test moves HEAD) before `bhangs` (its build hangs).
    It asserts the call ends within 30 seconds, where the build timeout is one minute, and that `amoves` is skipped with 2 queued.
- **F4** (commit `09962572`): fixed as proposed.
  - `internal/gatecli/doc.go` now says each test runs in its own process with its own environment and profile, all in the package directory, so a test that writes a fixed path there can disturb another.
  - It adds that the re-run alone bounds a failure or timeout beside others, but a run that still passes is recorded as it ran.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- One naming nit from fix-code-correctness, not a finding: the subtest name `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` now also covers the overlap row.
  The review asked for a row in an existing subtest, so the name stays.
- One note from fix-affected-set, not a finding: the `targets_test.go` row name "binary builder" is a plain description, left as it is.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
