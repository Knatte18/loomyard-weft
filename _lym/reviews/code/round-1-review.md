---
verdict: APPROVED
---

No kept findings. The diff makes the change the decision record settles, the declared `PATTERN-shed-recipe-registry` change lands, and no commit carries a `Pattern-Departure:` trailer.

What the review settled against the focus directive:

- `Expand` rewrites nested blocks innermost first, and each D2 refusal has a code path and a `TestExpand` row.
- `Parse` carries every D1 refusal and allows an empty `on_done` on a block row.
- Every router and reconstructor of a skipped entry reads `skipTarget()`: `enter` and `gotoAdmitted`, with `validate` refusing a `SkipTo` that names no producer.
- A goto onto a block row sets the entered row and so bypasses the gate; a re-launch clears it and evaluates the gate. Both paths are the generic ones, so no block-specific test row was required.
- Every `HistoryEntry` writer in `internal/shedengine` and the goto and re-launch entries carry `Block`, which is `omitempty`.
- `Block` is one `Constructor` entry returning Done, and `BlockGates` is a closed table reached through `LookupBlockGate`. `Build` refuses an unknown `block_gate` naming the row and the shipped checks.
- `internal/loomrecipe` reads every embedded recipe through `parseEmbeddedRecipe` and the one `embeddedRecipes` table, and `ProfileRecipe` serves the only reader of a task's profile block, `prepareRunSettings`.
- Motif and arras are pinned by `TestBlockRecipes`. Motif is absent from `shedrun.RecipeNames`, arras is present, and the tapestry recipe file is unchanged.
- Arras is accepted at each site that names tapestry. The remaining `RecipeTapestry` uses in `loomcli` are the no-seed fallback, and the bouncer and finalize lookups answer for arras because its rows keep tapestry's names and engines.
- The progress count needs no change in `progress.go`: the main line follows `OnDone` into the block, and a pass-over lands on the exit's later step.
- `_lym/landing/summary.md` does not exist, so there is no claim to check.

## Rejected

- Lens code-correctness F1 and lens code-affected-set F1 (`internal/shedverbs/human.go`, `blockAwareLast`), identifying a block row by `SkipTo != ""` and matching the activity line by `strings.HasPrefix`: not a defect. The told routing projection carries no engine, so `SkipTo` is the only block-row marker available there, and only `Expand` sets it. The prefix splice is a hypothetical about a row name that is a prefix of another's, with no such pair in any recipe.
- Lens code-correctness F2: the same `SkipTo` proxy as above, dropped with it.
- Lens code-tests: no findings were returned.
