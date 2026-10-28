# Fixer-report — code review round 1

## Changes

- F1 (fixed as proposed, commit `d2714d29`): `internal/reedengine/doc.go` now cites `internal/reedcli/smoke_lifecycle_windows_test.go` as the home of `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand`.
- F2 (fixed with own wording): the `PATTERN-test-speed` entry in `PATTERN.md` carries a no-OS-skip clause on its single line with the background link intact.
  The proposed wording "a skip names a missing prerequisite, never the OS" (commit `ff8954a3`) pushed the entry rule text to 287 characters, over the 280 cap that `TestOverview_RealFilePassesCheck` enforces.
  The clause is now "a skip never names the OS alone", which fits the cap.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `internal/loomcli/smoke_test.go:4` cites `smoke_lifecycle_test.go` as a convention example; the file still exists, so the reference holds. Disposition: no change, premise false (the file exists and the citation is general, not about the moved test).
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
