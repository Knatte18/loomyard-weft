---
round: 3
focus:
  - "Card 5 adds a `SamplerSession` row to the `Constructor` map in `internal/shedrecipe`. Find every test in the packages the card gate covers that hard-codes the full registry name list or a per-recipe expected list (for example `TestNames` in `internal/shedrecipe/registry_test.go`, and any table over `RecipeEngines` or `LoomBuildEngines`). Settle whether some card lists each one as an **Edit:** target and states the new expected entry."
  - "Card 6 maps `shedrun.RecipeSampler` in `embeddedRecipeSource` in `internal/loomrecipe`. Find every doc comment in `internal/loomrecipe` that enumerates the loom recipe names (the function and type docs in `names.go` and `doc.go`). Settle whether each is a listed target of the card or of another card."
  - "Card 7 widens `childRecipeSupported` and the halt-escalation helpers (`restepSkips`, `EscalateJudgedFirst`) to admit sampler. Find every doc or field comment in `internal/battencli` and `internal/battenshed` that lists tapestry and darn as the recipes covered, and settle whether the symbol carrying it is a target on some card."
  - "Card 3 describes a sampler stencil. Settle whether any path or filename the Intent has the stencil name would be refused by `TestStencils_NoBareCrossRepoCitations` in `contracts/stencils/citation_enforcement_test.go`, and whether the card handles it."
  - "For each card, compare the `**ImpactSummary:**` line with the functions the `**Intent:**` edits: the count of rows or callers, the production versus test-only change, and any extra file the Intent edits. Settle whether the summary states a conclusion the Intent contradicts."
---

Targets the defects round 2 raised, phrased as questions about the plan as it stands now.
