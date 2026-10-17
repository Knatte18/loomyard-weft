---
round: 1
focus:
  - "internal/reportengine/timingspans.go: settle whether the interval closed by a change out of `paused` is attributed to `paused` in every case, including a pause that spans UTC midnight, consecutive pause entries, and a run with no batten source."
  - "internal/reportengine/timingspans.go: settle whether the in-flight trailing interval under `paused` is bounded by the snapshot and cannot double-count against the producer's own phase."
  - "internal/reportengine/timing.go: settle whether active seconds for runs and buckets subtract the union of waits and the paused phase once, with no double subtraction when a wait overlaps a pause."
  - "internal/reportengine/timing_test.go: settle whether the two new `TestComputeTiming` rows duplicate an existing row's coverage, and whether any behavior the diff adds (phase order after `land`, median, bucket table) is left uncovered."
  - "internal/reportengine/doc.go and docs/overview.md: settle whether the described `paused` phase and its position in the phase table match the code, and whether `_lyx/landing/summary.md` claims anything the diff does not bear out."
---
Round 1 focus: attribution correctness of the new phase, the overlap union, and test economy against the board entry.
