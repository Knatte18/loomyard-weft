```yaml
lens: fix-affected-set
outcome: reported
```

I reran the set searches for F1, F2, F3, F5 and F6. F4, F7 and F8 each concern a single site, so they have no set to rerun.

Unchecked: none.

1. **LOW.** Card 7's new rows for `TestBurlerReviewProducer_Call_ProfileAndOpts`, added under "a `Rubric` seam whose answer differs…" and "a failing `Rubric` seam…".
   - **Summary:** the test builder these rows need is not on the card's **Edit:** list.
   - **Evidence:** `internal/shedadapters/burlerreview_test.go:65`, `newReviewProducer`, builds `BurlerHalfDeps` field by field from `halfProducerSpec` (lines 25–55), and that spec has no `rubric` field. Both the spec and the builder need a new field before either row can set the seam. Card 7 lists neither `internal/shedadapters#newReviewProducer` nor `internal/shedadapters#halfProducerSpec`.
   - The other two callers of `NewBurlerReviewProducer` are fine. `internal/shedrecipe/entries_burler.go:162` is `burlerReviewEntry`, which is already listed. `internal/loomcli/smoke_attachprobe_test.go:195` is fine with a nil seam.

2. **LOW.** Card 10's new sentence: "Every text that says PR-Review's Stuck is always a Halt … the header comment of `darn-recipe.yaml` and `sampler-recipe.yaml`…".
   - **Summary:** the tapestry header's no-on_stuck paragraph is not in the set this sentence names.
   - **Evidence:** `contracts/recipes/tapestry-recipe.yaml:29-31` lists PR-Review among the rows that "carry no on_stuck … and their Stuck halts the whole run". That statement goes false for PR-Review's send-back reroute in the same way. The sentence names only the darn and sampler headers. The older "no edge out" sentence may cover it, but the new sentence does not say so.

Members checked and covered:
- **F1:** the other hand-built `driftengine.Deps` cannot reach the new required seams.
  - `internal/shedrecipe/fixture_test.go:299` (`newTestEnv`) fills only a partial Drift set, with no `RecordsChangedSince` or `EndRoles`, so it already cannot build `DriftVerify`. `driftTestEnv` replaces it.
  - `internal/driftcli/wiring.go:200` never builds `DriftVerify`.
  - `internal/loomcli/driftwiring.go` and `envkit` are on the card.
- **F2:** `progress_test.go:73` is the only test that pins PR-Review's `OnReroute` as empty. `darnrecipe_test.go:203` checks only PR-Review's `OnDone`, `OnStuck`, `Segment` and `EntryGate`. `samplerrecipe_test` and `recipe_test` pin `OnReroute` only on Finalize.
- **F5:** `shed-recipe-spec.md:110` is the only markdown site. The `BouncerConfig.ReviewInput` passages in `internal/shedadapters/doc.go:293,306` are covered by card 7's `doc.go` line.
- **F6:** the sites match the earlier grep: `loomrecipe/doc.go:25`, the darn and sampler headers (`darn-recipe.yaml:30`, `sampler-recipe.yaml:43`), and the row comments at `tapestry-recipe.yaml:522`, `sampler-recipe.yaml:240` and `darn-recipe.yaml:278`. All are named except the item in finding 2.

Tool calls:
1. Grepped Go files for `driftengine.Deps{` builders and test files for PR-Review routing pins, to rerun the F1 and F2 searches.
2. Grepped markdown for `ReviewInput` and found the callers of `NewBurlerReviewProducer`, to rerun the F5 and F3 searches.
3. Read the `fixture_test.go` and `driftcli` Deps builders and the `bouncerrows`/`escalation` PR-Review rows, to judge whether those F1 and F2 members are affected.
4. Read the `newReviewProducer`/`halfProducerSpec` builder and every `OnReroute` pin, to check the F3 test set and the rest of F2.
5. Listed `newTestEnv`'s Drift seams and looked for `DriftVerify` construction in `driftcli`, to settle whether those F1 members break.
6. Grepped `doc.go` files and docs for the retired `ReviewInput` name, to finish the F5 set.
