# Round 1 fixer-report

## Changes

- F1 (MEDIUM): fixed as proposed.
  `internal/battencli/beamland_integration_test.go` gains the subtest `GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark` in `TestBeamLanding_ReachesWorktreeCreateSkip`.
  Before the mark, `hubStartGuard(h.Location, "g").refuse("g", "run", runShed, true, true)` must return an error carrying "grouping project".
  After `shedrun.WriteFilingMark(h.Location, "g")`, the same call must return nil.
  The tier-1 gate passes and shows the subtest as PASS.
  Commits: `4e3a6b5c`, plus one that names the new case in the file header (self-check LOW).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the file header omitted the filing-mark case; fixed by adding a header line.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
