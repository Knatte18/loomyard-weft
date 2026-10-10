# Card 13 — batten-admits-darn

**Create:**
- `plan:internal/battencli#childRecipeSupported` -> `func childRecipeSupported(recipe string) bool`
- `plan:internal/battencli#unsupportedChildRecipeRefusal` -> `func unsupportedChildRecipeRefusal(recipe string) error`

**Edit:**
- `internal/battencli#childSeedParams`
- `internal/battencli#battenCLI.wire`
- `internal/battencli/wire_test.go`
- `internal/battenshed#ErrUnsupportedChildRecipe`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D2: batten seeds a darn child through the same bootstrap.
`childRecipeSupported` admits `shedrun.RecipeLoom` and `shedrun.RecipeDarn`, the two recipes `lyx loom start` serves;
`unsupportedChildRecipeRefusal` wraps `battenshed.ErrUnsupportedChildRecipe` with a message naming both admitted recipes in place of today's loom-only text, and `wire`'s `WriteSeed` seam uses the pair.
`childSeedParams` writes `params.parent` for both recipes, since both bootstraps expect it.
The `Run-Shed` spawn stays `lyx loom start --no-attach`.
Tests: the write-seed refusal test gains rows for a darn child admitted and an unknown recipe refused with both names, and the params test covers darn.
The sentinel's doc names the two recipes.

**ImpactSummary:** Batten's child admit set gains one recipe; the spawn and every other row are unchanged.
