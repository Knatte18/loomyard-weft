---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1, commit `55075c5fc`: the spec bullet, the discussion template and the discussion-review rubric now read "the test that fails on the merge base, why no test seam exists, or that the entry fixes no bug".
- F2, commit `013410517`: `internal/gateslot/redfirst.go`, `internal/loomshed/doc.go` and the `holdsForCommitOrRecord` doc comment name every record that lifts the gate, not the No test seam record alone.
- F3: `_lym/landing/summary.md` names loomshed, gatecli and the darn writer stencil as the blast radius.
  The file lies under `_lym`, so the edit is not committed.

- Gate repair: the gate found no mapped test reaching `redFirstCall.run`.
  `redFirstCall` gained a `runTiers` field, set to `runTest` in `runRedFirst`, so a tier-0 test can stand in for the tier runs.
  `TestRedFirstCallRun_JudgesTheTierRunsOfTheChangedTestsOnTheBase` pins the red, green (both ways forward) and unjudged verdicts.
  The loomshed reach lines name tests that already exist and need no change.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `internal/gatecli/doc.go:472` still names `NoTestSeamTitle` alone as the way forward of the no-test and unjudged exits.
  Disposition: disputed, premise false.
  The sentence states the way forward the finding text names and claims no exclusivity.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- fix-consistency fork finding on `internal/gatecli/doc.go:472`, ground `premise false`: the sentence says "The way forward of both is a recorded decision whose title opens `gateslot.NoTestSeamTitle`".
  It names a way forward and does not say only that record lifts the gate, so the "only" premise does not apply.
  Evidence: `internal/gatecli/doc.go:472`, `internal/gatecli/redfirst.go:42`.

## Deferred

None.

## Rulings

None.
