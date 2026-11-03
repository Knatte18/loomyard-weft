```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review of the plan's cards: is each new guard, refusal and test row reachable, and does each guarded site recur unguarded elsewhere?

1. MEDIUM — Card 8 (D8, failed-commit refusal): the pointer at `lym loom decision store` is added only on the no-amendment path, but the same guarded site recurs unguarded on the `--amends-plan` path. On a run past its approval, `afterFailedCommit` and `rollBackAppend` still send the caller to `lym fabric commit` alone. That lands the record and never copies it. Evidence: `internal/loomcli/decision.go:583`, `:589`, `:593` (inside `internal/loomcli#amendmentCall.afterFailedCommit`) and `:566`, `:570` (`rollBackAppend`) all end in `decisionWayFabricCommit`. Every `--amends-plan` run is past the approval, since it needs an approved plan (`decision.go:712-721`). Card 8 changes only the `commitErr != nil && amendment == nil` refusal at `decision.go:783-785`.

2. MEDIUM — Card 8 (`copyRecordToBoard`): the card names no behaviour, and no test row, for `deps.pastApproval` returning an error. It only says "returns "" when pastApproval is false". The branch is reachable in `preRun`'s wiring: `PastDiscussionApproval` returns an error when the recipe name selects no embedded recipe. A silent skip, a `record_store_error` or a refusal are all possible readings. Evidence: Card 8, "The copy"; `internal/loomrecipe#parseEmbeddedRecipe` refuses an unknown name (`names.go:111`).

3. MEDIUM — Card 9: the refusal "the run has no committed decision record …" has no test row. The status-read and recipe-read refusals (re-run `lym loom decision store`) and the "parent cannot be resolved" caller branch have none either. Each can occur through the fakes: `committedFound=false`, `readStatus` error, `parent` error. The rubric asks for a named reaching test for every prescribed refusal. Evidence: Card 9, "The copy", "Refusals" and "Callers", against the rows of `TestDecisionStoreVerb`.

4. LOW — Card 6: no recipe or test row reaches the `discussionApprovalRow` branch where a DiscussionWrite row hands to a non-Bouncer row. The test covers only tapestry (which hands to a Bouncer), darn and pawl (which have no DiscussionWrite row), and Card 7's fixtures also hand to a Bouncer. Evidence: Card 6 test rows; `contracts/recipes/tapestry-recipe.yaml:115-117` is the only DiscussionWrite row.

5. LOW — Card 8: the "missing committed record or read error → error + `decisionWayStore`" branch of `copyRecordToBoard` has no row. Evidence: Card 8, "The copy", against the rows of `TestDecisionVerb_CopiesRecordToBoard`.

6. LOW — Card 8: the "beam's project record, nothing is stored" row cannot reach the guard it names. The row sets the fake's `pastApproval` to false, so it repeats the "inside Discussion-Review" row. The real reason a beam is never copied is `PastDiscussionApproval("pawl")` answering hasRow false in `preRun`, and no test runs that through `decisionVerb`. Evidence: Card 8 tests (`decisionFake` gains `pastApproval bool`); Card 6 covers pawl only at the loomrecipe level.

7. LOW — Card 2 (failure path): a failed record move or removal returns from `boardCriticalSection` after the save, so the render and the sync spawn never run. The saved rename then leaves the old design file and the README stale, and nothing syncs the rename until the next write. The guarded failure state is reachable (Card 2's directory-target row), but the card neither states this effect nor asserts the render. Evidence: `internal/boardengine/board.go:156-168`, where an `afterSave` error returns before `RenderToDisk` and `spawnSync`; Card 2 puts its step after `afterSave` and before the render.

Tool call: one grep of `internal/configengine` for how an empty scalar value is handled. It was for Card 3's empty-`design_prefix` row, which my context could not confirm. The grep was inconclusive, so I raise no finding on that row.
