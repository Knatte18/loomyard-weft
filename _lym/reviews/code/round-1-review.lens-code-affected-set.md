```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I looked for the callers, readers and assertions it forces outside the diff.

**Per change: the set found, and whether it is covered**

- **`SkipTo` and `skipTarget()` (skip routing).**
  - Set: `enter` in `entry.go`, `gotoAdmitted` in `goto.go`, `validate.go`.
  - All three read the skip target, so they are covered. The reader in `entry.go` was the only one that routes a skipped row.
  - The other readers of `OutcomeSkipped` in `run.go`, `activity.go`, `settledguard.go`, `checklist.go` and `timingspans.go` only classify an entry and do not route it. The remaining `.OnDone` reads, in `run.go` (the Done path), `samplerwiring.go` and `settledguard.go`, are not skip routes. Covered.
- **`HistoryEntry.Block`.**
  - Set: shed-appended entries in `run.go`, `entry.go` and `reroute.go`, plus the goto and relaunch entries in `goto.go`.
  - All carry `def.Block` or `target.Block`. The field is `omitempty`. Covered.
  - `status.go` is covered by the added `status_test.go` change. Covered.
- **`ProducerDef` and `Routing` gain `SkipTo` and `Block`.**
  - Set: `shedbuild` Build, `RoutingOf`, `RoutingFrom`, `loomrecipe` Routing and Rows, and `shedverbs` human.
  - Covered.
  - `shedcheck/check.go` walks only OnDone and OnStuck, so it never sees the skip edge. No finding: nothing in the diff passes an expanded loom recipe to `shedcheck`.
  - `progress.go` `mainLine` follows OnDone into the block's rows, so a block row counts as its own step and the following steps are numbered through the inlined rows. Covered by the added `loomrecipe/progress_test.go`.
- **The Block engine and `BlockGates` table.**
  - Set: the registry entry, the cross-consumer conformance guard (the `conformance_test.go` change) and the `loomshed` `InterruptPolicies` row. All covered.
  - `PATTERN.md` carries the declared text for `PATTERN-shed-recipe-registry`, and the `pattern/` background file gained a "Block gates" section. The declared change landed.
- **Embedded recipes expand in one place.**
  - Set: `RecipeEngines`, `Rows`, `AmendmentTarget`, `bouncerrows.go`, `finalizerow.go`, `Routing`, `New`, `ProfileRecipe`.
  - These all go through `parseEmbeddedRecipe`.
  - The other direct parses of an embedded recipe are `recipes.BattenRecipe` in `beamcli/dobbydeps.go:251`, `battenrecipe/battenrecipe.go:47` and `battenrecipe/names.go:60`. The decision record says batten and pawl name no block, so these are out of scope. Covered.
- **Sites that name tapestry.**
  - Covered (`arras` is accepted at each): `battencli` `wire.go` and `runsettings.go`, `reportengine` `runs.go` and `timingspans.go` (with a phase for `Motif`), `shedcli/table.go`, `shedrun` `RecipeNames`, `tools/tokencount`.
  - The `loomcli` default-to-tapestry sites (`arm.go:124,344,355`, `peers.go:88,260`, `sharedbootstrap.go:208`, `wiring.go:1031`) are only the fallback for a run with no seed yet.
  - `RetiredRows` is keyed on tapestry only, which is harmless because `arras` has no older status files.
  - `finalizerow.go` and `bouncerrows.go` loop over tapestry and darn only. The `arras` rows have the same names and engines as tapestry's, so the lookups still answer. Covered.
- **The `block` field on history and the block-aware status display.**
  - Set: the checklist (`checklist.go`) and `human.go`.
  - Both are covered by the added `checklist_test.go` and `human_test.go`.
  - `human.go:877` treats a row as a block row because its `SkipTo` is non-empty, not because its engine is Block. This is a weak identification; it holds today only because `SkipTo` is set solely on block rows. See the finding below.
- **Documents.** The changes in `shed-recipe-spec`, `loom-status-spec`, `docs/overview.md` and the package `doc.go` files all follow the code. Covered.
- **Commit messages.** None carries a `Pattern-Departure:` trailer, so none is checked.

**Findings**

1. LOW, consistency, `internal/shedverbs/human.go:877`. `blockAwareLast` identifies a block row by `def.SkipTo != ""`. That is an indirect proxy, and `ProducerDef.SkipTo`'s own comment lets any row set it ("a row whose skip must jump past the rows its Done enters"). `loomrecipe` identifies it by `Engine == EngineBlock` (`IsBlock`). A non-block row given a `SkipTo` would then read "→ entered". Fix: give the routing projection an explicit block-row marker, or document in the `ProducerDef.SkipTo` comment that only block rows may set it.

No other member of the affected set is left wrong.

**Tool calls**
1. grep for `OutcomeSkipped` and `.OnDone` readers outside the diff: my context lacked the other skip and route readers.
2. grep for the direct `shedbuild.Parse`, `RoutingOf` and embedded-recipe parses: to find parses that bypass expansion.
3. Read `shedengine/progress.go`: the diff showed only a comment change, and I needed to see that `mainLine` follows OnDone.
4. grep in the progress tests and `shedcheck`: to see whether either reads the skip edge.
5. grep for `RecipeSampler` without `RecipeArras`: to find recipe lists that name recipes but not `arras`; it found none.
