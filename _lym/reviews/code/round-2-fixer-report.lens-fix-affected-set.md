```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for F15 (callers of `DarnTable` and `MeasureCall`) and F17 (sites that read `Covers`). I found no findings.

- **F15 callers:**
  - Every `DarnTable` caller is covered. `darnengine/spec_test.go` and `wiring_darn_test.go` were updated, and `wiring_test.go` was fixed after the gate run. `shedrecipe/entries_darnwrite.go` only forwards the seam, so it needs no change.
  - `shuttleparity/seats.go:201` always passes `measureCallStandIn`, so it builds under both the new conditional rule and the old one.
  - The only test that asserted "gate config absent" was the `wiring_test.go` row. The fixer corrected it and removed the dead `wantTableErr` field.
  - No other test or doc depends on an ungated table failing for a missing or invalid gate config. `docs/` and `contracts/` mention `measure_call` only for the gated fragment, which stays consistent.
- **F17 sites:** The only other `Covers` readers that decide which tiers a call runs are `gatecli/tiers.go` (`dropCovered` at line 340 and the explicit-target check at line 154). `tiersRunBy` mirrors `dropCovered`: it works from the highest tier down, keeps a tier unless a kept tier's `Covers` holds it, and sorts and compacts the list first. `hardenwiring.go:322` and `gatecli/green.go:91` read `Covers` for other purposes and are unaffected.

Tool calls:
1. A grep for `DarnTable(`, `DarnInputs{`, `measure_call`, `MeasureCall`, `gate config absent` and `Covers` across the repo. My context lacked the full caller and site sets.
2. A read of `shuttleparity/seats.go` lines 195–225, plus a grep for leftover `wantTableErr`, `gate config absent` and `no {args}` assertions. My context lacked that stand-in and those test sites.
