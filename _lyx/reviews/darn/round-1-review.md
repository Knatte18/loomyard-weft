---
verdict: APPROVED
---

No findings.

The focus questions, settled:

- The interval a change out of `paused` closes counts under `paused` in `phaseOf`, including across UTC midnight, for consecutive pause entries, and with no batten source (no pause is recorded, so the run times as before).
- The in-flight trailing interval under `paused` runs from the latest stamp to the snapshot and is added once, so it cannot double-count against the producer's phase.
- Active seconds for runs and buckets subtract `inactive`, the union of net waits and the paused parts.
  The paused parts are already net of sleep, so sleep is not subtracted twice, and a wait that overlaps a pause is subtracted once.
  The new row's figures check by hand: active 2260 for the run, 1260 and 1000 for the two day buckets.
- The two new `TestComputeTiming` rows duplicate no existing row, and each pins a distinct branch.
- `doc.go`, `docs/overview.md` and `_lyx/landing/summary.md` match the diff.

## Rejected

- code-tests LOW: no test pins `paused`'s position after `land` in `timingPhases`. The position is one literal slice entry with no logic, so a test on it is a mirror of the literal and would add coverage the repo's test economy does not ask for.
- code-correctness note: a loom row sharing a stamp with the pause entry could hide a standing pause. It depends on an unlikely tie and on `mergeRows` ordering that the diff does not change, so it is not a defect of this diff.
