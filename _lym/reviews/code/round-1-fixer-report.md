---
dispositions:
  - id: F1
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer report, round 1

F2 is `decision`-class and held by the parent, so it is not in this report.

## Changes

- F1, commit `2afa0db1e`, plus a follow-up commit for the line break below:
  the reflection stencil, the `loomcli` package doc and the `AnomalyBuildSkew` doc state the reviewer's wording.
  The driver hands such a run to a fresh driver when a newer build is on disk;
  the "back to its parent" alternative is gone.
- F3: `_lym/landing/summary.md` now says the driver does not hand the run over and its tackler settles the stop.
  Not committed, since no git command runs against `_lym`.
- F4, commit `2a416f460`: `TestClassifyHalt` gained the row `NotFailedBuildSkew_NoKind`.
  `go test` through `lym gate test ./internal/loomengine` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: LOW, the F1 stencil sentence ran past 119 characters. Fixed as proposed by breaking the line after "it read,".

## Disputed

None.

## Deferred

None.

## Rulings

None.
