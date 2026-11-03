```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and new branch in the plan for a card and a test row that would fail if it broke.

**Findings**

1. **MEDIUM — D3 and D6, `lym loom status` (card 9).** The `lym loom status` verb itself never shows the block.
   - The verb renders through `shedverbs.RenderStatusHuman` (`internal/shedverbs/status.go:219`, `internal/shedverbs/human.go:29`). It prints progress, `now` and `Activity.Last`.
   - `Activity.Last` is composed by `composeActivity` as `"<producer> → <outcome>"` (`internal/shedengine/activity.go:35`), so it reads `Motif → done`, not entered. No block appears beside the row name either.
   - Card 9 changes only the checklist pane, `renderChecklist` (`internal/loomcli/checklist.go:200`), which the hidden `checklist` verb draws.
   - No card and no row covers the criterion "`lym loom status` shows as entered … shows it beside the row's name" for the verb.
   - Fix: carry the entered word and the block mark in `RenderStatusHuman`, or in `composeActivity`, with a `TestRenderStatusHuman` row. Otherwise state in card 9's Intent why the pane is the surface the record means.

2. **MEDIUM — D11 criterion "the loom build, the routing projection and the row and engine accessors return the same expanded rows for arras, block path and skip target included" (card 6).**
   - No planned row compares `loomrecipe.New(arras)`'s `ProducerDef.SkipTo` and `Block` against `loomrecipe.Routing(arras)`'s.
   - `TestBlockRecipes` compares parsed expanded rows only. `TestRowNames_FollowEachRecipesRowOrder` covers `Rows`. `TestRoutingOf_MatchesNewShedRowForRow` (card 5) covers a generic shedbuild recipe, not the loomrecipe path.
   - No row checks that `HasEngine` or `EngineRow` see the expanded arras rows. One example: `EngineRow(arras, "DiscussionWrite")` should be `Discussion-Write`.
   - Fix: add an arras subtest that compares New against Routing row for row, `SkipTo` and `Block` included, plus `TestRecipeKeys` rows for `HasEngine` and `EngineRow` on arras.

3. **MEDIUM — D7 criterion "the Discussion-Review segment's budget comes from the discussion perch keys" (card 6).**
   - `TestReviewBudget_AppliedByNewAndRouting` for arras only checks that a told `Env.Segments` or budgets-map entry reaches the rows.
   - The mapping from the `Discussion-Review` segment to `loom.yaml`'s `discussion_*` perch keys is loomcli's segment-to-perch wiring (`loomSegmentPerches` in `internal/loomcli/wiring.go`). No arras row reaches it.
   - Fix: add an arras row to loomcli's recipe-keyed wiring test (`TestRecipeKeyedWiring`), asserting that Discussion-Review resolves from the discussion perch keys.

4. **LOW — D2 refusal list (card 4, `TestExpand`).**
   - The plan promises "one refusal row per bullet", but several bullets hold more than one case:
     - unknown recipe vs nil resolver;
     - self-inclusion directly vs through its own blocks;
     - `on_done` naming itself vs naming an inlined row;
     - the four run-only keys `contract`, `verify_source`, `amendment_target` and `profile_recipe`;
     - `on_done`, `on_stuck` and `on_reroute` leaving the recipe.
   - One row per bullet leaves the other cases of each bullet unreached. The nil-resolver case and the indirect self-inclusion are separate branches.
   - Fix: give each case its own row.

5. **LOW — card 2, `LookupBlockGate`.** The Intent refuses "an unknown or empty name", but `TestBlockEntry` plans only the unknown-name row, so the empty-name branch goes untested.

6. **LOW — card 1, gate bypass.** No row pins that a goto onto an inlined row of a passed-over block is admitted. D3 says such a move "runs that row with the block gate never held".
   - `gotoAdmitted` admits by list index (`internal/shedengine/goto.go:338`). An inlined row lies before the exit in list order, so it is admitted.
   - The plan covers only goto and relaunch onto the block row itself in `TestStep_Block`.

7. **LOW — D11 fidelity (card 6).** The record says "The batten and pawl recipes name no block and their loaders tell no resolver".
   - Card 6's `parseEmbeddedRecipe` tells `embeddedRecipeSource` as the resolver for every recipe, batten and pawl included.
   - The outcome is harmless, since `Expand` returns a recipe with no block unchanged. Still, the card departs from the record's text without recording why in Shared Decisions.

8. **LOW — D2 "A sub-recipe's `terminals` are not read" (card 4).** No row pins it.
   - Example: a sub-recipe whose `terminals` names a row outside it, or whose terminals differ from the parent's, should still expand.
   - The parent's `terminals` should stay the run's.

9. **LOW — D4 "the review rings inside the block then work as in the sub-recipe's own run".**
   - `TestNew_ShapeMatchesRecipe`'s ring and reachability checks run only on tapestry (`internal/loomrecipe/recipe_test.go:28`).
   - `TestBlockRecipes` only asserts that `New(arras)` builds, so arras's inlined Discussion ring shape (OnStuck chain, equal Segment and MaxBounces) is not asserted on the built list.

Tool call made:
- One Bash grep and sed. I checked which renderer `lym loom status` uses, `settledguard.go`'s `OnDone`-based predecessor walk, that `TestShed_Validate` exists, and the recipes used in `TestPrepareRunSettings`. My context had none of these files, only the subagent summaries.
