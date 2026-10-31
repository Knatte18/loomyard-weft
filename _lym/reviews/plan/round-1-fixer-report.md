# Fixer report — plan, round 1

Every finding's premise held against the code; all seven are fixed.
`lym loom validate-plan` reports `ok: true` with informational findings only.

## Changes

- F1 (BLOCKING), fixed differently.
  Proposed: move the recipe file, `SamplerRecipe`, the `RecipeEngines` reach and its test to card 5, with the recipe-content description staying in card 6's Intent and card 5 pointing forward at it.
  Not done: the forward pointer, since card 5 would write a file whose contents only a later card describes, and an implementer working card by card would build it from a card not yet reached.
  Done: card 5 creates `contracts/recipes/sampler-recipe.yaml` and `plan:contracts/recipes#SamplerRecipe`, edits `internal/loomrecipe#RecipeEngines` (sources and its "three recipes" doc sentence) and `#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`, extends `contracts/recipes/recipes.go`, uses `contracts/recipes/darn-recipe.yaml`, and carries the recipe's row and comment description moved from card 6, opened by the sentence that it does so for `TestCoverageGuard`.
  Its ImpactSummary and Commit subject name the embedded recipe.
  Card 6 drops those Create, Edit and Extends lines and its darn-recipe Uses, uses `plan:contracts/recipes#SamplerRecipe`, and its Intent says the file is card 5's.
  `00-overview.md`'s index lines for cards 5 and 6 follow.
- F2 (MEDIUM), fixed as proposed: card 8 edits `internal/landingshed#Deps`, uses landingshed's `doc.go` and the seven Env-reflecting seam tests, and its Intent states the new `CodeReviewOff` and `RerouteReviewed` docs.
- F3 (MEDIUM), fixed as proposed: card 3's `TestSessionSpec` no longer asserts the stencil's fixed wording.
- F4 (LOW), fixed as proposed: card 7 lists `internal/landingshed/handback.go` in place of `#EscalateHalt`.
- F5 (LOW), fixed as proposed: card 12 edits `internal/loomengine/template.yaml` and its Intent states the `step_idle_timeout_min` comment.
- F6 (LOW), fixed as proposed: card 6 edits `internal/loomrecipe#RowNames`, and its Intent has the `Rows` and `RowNames` docs name `shedrun.RecipeSampler`.
- F7 (LOW), fixed as proposed: card 9's ImpactSummary names only `wire` and `wireLanding`.

## Self-check

Duplicates are merged under the first lens that raised them.

- fix-consistency, MEDIUM, F3's cut leaves the D8 and D9 wording criteria with no planned check (merged with fix-proposal MEDIUM): fixed as proposed (the fork's first option) — card 3 states the criteria are met by the fixed stencil text the card writes, checked by reading it at review, and why no test pins them.
- fix-consistency, LOW, card 5's Commit subject omits the embedded recipe: fixed as proposed.
- fix-consistency, LOW, card 6's `darn-recipe.yaml` Uses is stale after the move: fixed as proposed — removed.
- fix-claims, LOW, `RecipeEngines`' "All three recipes" doc sentence left unnamed (merged with fix-affected-set LOW on the same line): fixed as proposed — card 5 says the sentence becomes four.
- fix-affected-set, LOW, `internal/loomengine/config.go`'s file comment and `loomCLI.stepIdleTimeout`'s doc also say an idle step is killed: fixed as proposed — card 12 edits `internal/loomengine/config.go` and `internal/loomcli#loomCLI` and says both docs state the exemption; `validate-plan` then required `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag` under Uses, which the card now lists.
- fix-affected-set, LOW, `loomrecipe/doc.go`'s Rows/RowNames and AmendmentTarget sentences list four recipes: fixed as proposed — card 6's Intent has them name sampler; the card already edits that file.
- fix-proposal, MEDIUM: merged into the first line above.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
