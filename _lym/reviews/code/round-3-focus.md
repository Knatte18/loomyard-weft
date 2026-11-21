---
round: 3
focus:
  - "internal/loomcli/darnwiring.go darnTable closure and internal/darnengine/spec.go DarnTable: does a darn run whose writer has no cold fixer still build its table, and so pass Preflight, on a hub whose darn-write tiers hold no {args}; and does a gated writer still refuse an empty measuring call by name."
  - "internal/gateslot/tiers.go Config.MeasureCall: for a key listing tiers where a higher listed tier's covers drops a lower one, does the rendered call match what internal/gatecli/tiers.go planTierRuns accepts, for both the target word and the {args} check."
  - "internal/gatecli/trx_test.go and internal/gatecli/tierrun_spawn_test.go: does each remaining test pin behaviour no other test pins, and does TestRunTierReadsTheVSTestResultsFile assert the result words from literals rather than through the production helper."
---

The focus lists the areas the round-2 fixes touch.
