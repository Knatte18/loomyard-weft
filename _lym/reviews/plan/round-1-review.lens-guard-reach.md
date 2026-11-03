```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: whether each guard, refusal, branch or test row the cards add can be reached through a real entry point.

**Per guard: reachable or not**

- Card 1, `Shed.validate` refusing a `SkipTo` that names no producer: reachable only by a hand-built list, through `TestShed_Validate`. It is a fail-loud wiring guard, so it is exempt.
- Card 1, `Shed.enter` skipping by `skipTarget()`, with an empty skip target finishing the run: reachable through a block row's gate not holding.
- Card 1, `gotoAdmitted` reading a skipped entry's route as `skipTarget()`: reachable. Both new rows of `TestGoto_BlockedRunAdmitsOnlyEarlierRows` reach it: `current_producer` naming no row is set by the fixture, and a run blocked at the exit admits the earlier inlined rows. No other reader rebuilds a skipped entry's route. `composeActivity`, the episode count in `run.go` and the checklist only test for `OutcomeSkipped`.
- Card 1, the `Block` stamp at every history write: reachable. Every `HistoryEntry{` site (`entry.go:51`, `run.go:352`, the three in `reroute.go`, `goto.go:136` and `goto.go:250`) is covered by the card.
- Card 3, `checkRowShape`'s refusals (a block row setting an engine key, a missing `block`, a missing `block_gate`, a hand-written `engine: Block`, and `block_path` written in a file): all reachable through `Parse`. Nothing ahead of `checkRowShape` rejects these inputs, and `KnownFields(true)` refuses a `yaml:"-"` key as unknown.
- Card 4, Expand's refusals:
  - unknown recipe, nil resolver, self-inclusion (direct and through an inner block), shared row name, shared segment: reachable.
  - inlined `on_done`, `on_stuck` or `on_reroute` naming no row of its recipe: reachable. `Parse` checks no routing (`parse.go:37-40`), so nothing upstream rejects the input.
  - outermost block row with an empty `on_done`: reachable, since card 3 admits an empty `on_done` on a block row.
  - block row whose `on_done` names itself or an inlined row: reachable, since `Parse` does not check it.
  - sub-recipe setting `contract`, `verify_source`, `amendment_target` or `profile_recipe`: reachable.
  - Guarded site that recurs unguarded: the sub-recipe's `entry`. See F1.
- Card 5, `Build` refusing an unexpanded block row ahead of the loom contract check: reachable. `Parse` admits a block row with an empty engine, and `NewShed` and `Load` callers reach it too.
- Card 5, `Build` refusing an unknown `block_gate`: reachable. `Expand` does not check the gate name, and `Parse` only requires it to be non-empty.
- Card 5, `RoutingOf` refusing a parsed block row: reachable through bytes that hold a block row.
- Card 2, `LookupBlockGate`'s unknown-name refusal: reached by card 5's `TestBuild_Errors` row.
- Card 2, the coverage allowlist entry for `Block`: reachable, and card 6 removes it.
- Card 8, `ProfileRecipe` refusing an unknown name: reachable through `TestRecipeKeys`.
- Card 8, `prepareRunSettings` keeping `settings.Recipe` when `ProfileRecipe` refuses, and its test row: **not reachable**. See F2.
- Card 9, the checklist with no readable recipe (empty block set): reachable.
- Card 9, `blockAwareLast`'s block-row branch: reachable. Every expanded block row carries a non-empty skip target, because the outermost exit is required to be non-empty.

**Findings**

- F1 — MEDIUM, class design, card 4 (`shedbuild#Expand`).
  - Summary: Expand sets the block row's `OnDone` to the sub-recipe's `entry`, and that edge is exempt from the edge-leaving-the-recipe refusal. Nothing checks that `entry` names a row of the sub-recipe.
  - Effect: a sub-recipe whose `entry` names a parent row (for example `entry: Drift-Baseline`) builds, and the run silently passes over every inlined row while the gate holds. An `entry` naming no row reaches `shedengine.validate`'s OnDone refusal, which carries no way forward. Card 4 should add an Expand refusal for an `entry` that names no row of its recipe file, with a `TestExpand` row.
  - Evidence: `internal/shedbuild/parse.go:80-82` (checks only that `entry` is non-empty); `internal/shedengine/validate.go:74-75`; card 4 lines 19 and 32.
- F2 — MEDIUM, class design, card 8 (`battencli#prepareRunSettings`, `TestPrepareRunSettings`).
  - Summary: the fallback branch and the test row "a task whose recipe names no embedded recipe" guard a state that cannot occur. Seed-Child calls `WriteSeed`, which refuses any name outside `shedrun.RecipeNames` and any recipe `childRecipeSupported` rejects, before `writeRunSettings` calls `PrepareRunSettings`.
  - Effect: every name that reaches `prepareRunSettings` is an embedded recipe, so `ProfileRecipe` never refuses there. The card's claim that such a task "reaches Seed-Child's seed write" has the order backwards. The swallow branch should be dropped and the error propagated (or treated as a wiring guard), and the unreachable test row removed.
  - Evidence: `internal/battenshed/seamchild.go:93` (`WriteSeed`) runs before `:141` (`PrepareRunSettings` inside `writeRunSettings`); `internal/battencli/wire.go:1392-1400`; card 8 line 18 and line 28.

**Tool calls**

1. `activity.go:20-45`, `run.go:636-655` and the `routedTo`/skipped spans of `checklist.go`: to check whether other readers rebuild a skipped entry's route. The manifest held only grep hits.
2. `ValidateRecipe` callers, `RecipeOf`, and the call sites of the battencli prepare and child-recipe functions: to trace whether an unknown recipe can reach `prepareRunSettings`.
3. `wire.go:1335-1405`, `boardengine` `recipeOf`, and board recipe validation: to see the Seed-Child closures and the order of the seed check.
4. battenshed grep for `PrepareRunSettings`, `WriteSeed` and `Recipe`: to find where Seed-Child sets the recipe and calls each one.
5. `seamchild.go:60-150` call order: to confirm `WriteSeed` runs before `writeRunSettings` and `PrepareRunSettings`.
