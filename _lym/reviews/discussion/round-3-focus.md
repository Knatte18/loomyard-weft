---
round: 3
focus:
  - "Decision record D9 step 3 and Bound: after step 3 reaches origin, can a halt or crash at step 4 or 5 still be resumed? Check the origin paths an entry marked done before the run ends meets: boardengine replay.go `Ended: isDone`, battencli `resumeRefusal`, pairteardown `teardownEnded`, and fabricengine `withoutFinalizeEntered`. Settle which marker step 3 leaves and when the run's own entry becomes done."
  - "D9 step 3 marker: does the marker survive a released part landing and being pruned (boardengine store.go prune frees slugs) before a retry? Settle whether the step 3 done check reads anything that prune removes."
  - "D9 steps 3 and 4 for shape beam: between the step 3 board write and step 4's `lym beam create`, can the parent level's dobby start a filed part (boardengine parentbeam.go `BeamUnder` falls back to `ParentBeamOf` while the project is no beam)? Settle the release order."
  - "D10 and D9 step 3: which commit stages the raddle store directory, and which replayer claims its paths (boardengine board.go `spawnSync` and replay.go `FabricReplay.Claims`, which refuses a path with a `/`)? Settle how the store reaches origin beside a concurrent board write from another machine."
  - "D12 against the declared `PATTERN-run-usage-record` (only shed's one usage writer records a session's tokens and cost under `_lym/usage`): `internal/shedadapters/roundusage.go` writes each review half's token reading to `round-<N>-usage.yaml`. Settle whether the round record keeps its own reading, folds onto the one writer, or is a reader, and whether the declared text matches."
  - "D12 pricing source: `SessionUsage` (`internal/shuttleengine/sessionusage.go`) carries session totals, while `PriceMessage` (`transcriptread.go`) takes one message's model and usage. Settle where the writer reads per-message data and what the record's model field holds when a session's forks run on another model."
  - "D5, D8, D10, D11 and D14 slug freshness: prune frees board slugs while the raddle store outlives prune and is looked up by slug. Settle whether a reused slug can collide with an older store entry."
  - "D6 and D8 checks: does a carried criterion or Open risk get checked against the decisions its part builds, and does D8's module-label re-check cover an entry created after the agent's board read and a note promoted after the baseline (boardengine dates.go keeps a promoted note's dates)?"
---

Round 3 reviews the changes made for the open design items above; a fresh reviewer should also read the whole record for new breaks the changes introduce.
