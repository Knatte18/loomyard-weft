# Card 13 — batten-admits-darn

**Create:**
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`

**Edit:**
- `internal/battencli#childSeedParams`
- `internal/battencli#battenCLI.wire`
- `internal/battencli/wire_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/battenshed#ErrUnsupportedChildRecipe`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D2: batten seeds a darn child through the same bootstrap.
`childRecipeSupported` admits `shedrun.RecipeLoom` and `shedrun.RecipeDarn`, the two recipes `lyx loom start` serves;
`unsupportedChildRecipeRefusal` wraps `battenshed.ErrUnsupportedChildRecipe` with a message naming both admitted recipes in place of today's loom-only text, and `wire`'s `WriteSeed` seam uses the pair.
`childSeedParams` writes `params.parent` for both recipes, since both bootstraps expect it.
The `Run-Shed` spawn stays `lyx loom start --no-attach`.
What bounds which runs carry a darn seed: batten's child seeding admits `darn` only from a board entry whose operator-set `recipe` is `darn`, and any other name keeps the refusal;
the one other writer of a darn seed is a hand `lyx shed seed`, and `lyx loom start` never rewrites a recorded recipe.
Tests: the write-seed refusal test gains rows for a darn child admitted and an unknown recipe refused with both names;
batten's four-row lifecycle step runs once per admitted recipe, asserting the child seed names the board entry's recipe and carries `params.parent` equal to the pair's recorded parent branch.
The sentinel's doc names the two recipes.

**ImpactSummary:** Batten's child admit set gains one recipe; the spawn and every other row are unchanged.
