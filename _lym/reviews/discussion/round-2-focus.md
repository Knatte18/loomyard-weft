---
round: 2
exclude_lenses: []
focus:
  - "In `_lym/discussion/decision-record.md`, D4's side-by-side count is the tier's `memory_mb` divided by `redundancy_test_memory_mb`: settle what the record says when a mapped tier's `memory_mb` is below `redundancy_test_memory_mb`, and whether the stated Bound that the tier's cap bounds the run's test processes together still holds under that rule."
  - "In D3 and D4 of `_lym/discussion/decision-record.md`, settle whether any test a map run executes can reach a `lym` build the run made, given that `internal/testkit/lymbin` is the only reader of `LYM_PREBUILT_LYM` and `internal/testcover/spawnscan.go` `ScanSpawns` marks every test reaching `lymbin` as out of process; check that every D3 criterion describes a test that can run."
  - "In D4, D6 and D7 of `_lym/discussion/decision-record.md`, settle for a package holding both tier 0 and tier 1 tests which tier's `parallel` and `memory_mb` bound the processes running at once and which tier's `-p` the coverage builds take, and whether D6's pre-write checks, D7's writes and D1's refusal list agree with each other on which maps are written and which flags are refused."
---

Focus targets the round 1 fix areas: D1, D3, D4, D6 and D7 of the decision record.
