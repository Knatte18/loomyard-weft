# Card 33 — batten-seed-beam

**Create:**
- `internal/battencli#boardRecipe`
- `internal/battencli#TestBoardRecipe`

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#childRecipeSupported`
- `internal/battencli#unsupportedChildRecipeRefusal`
- `internal/battencli#childSeedParams`
- `internal/battencli#TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree`

**Uses:**
- `internal/shedrun#RecipeBeam`

**Intent:**
D11: batten runs its rows over a beam's landing as over a task, and Seed-Child seeds recipe `beam` for a project entry.
`boardRecipe` answers the recipe Seed-Child seeds: `beam` for a project entry, the entry's own `recipe` field otherwise, and Seed-Child's `ReadBoardType` seam in `wire` returns it.
`childRecipeSupported` admits `beam`, since loom's bootstrap verb runs it (card 32), and `unsupportedChildRecipeRefusal` names the three admitted recipes; `childSeedParams` gives a beam seed the same `parent` param from the pair's origin record, the beam's parent branch.
`TestBoardRecipe` rows: a project gives `beam`, a task with `recipe: darn` gives `darn`, a task with none gives the empty recipe.
`TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree` asserts the refusal names all three admitted recipes, and gains a `beam` row that passes the admission and reaches the absent-pair refusal as `loom` does, the point where `childSeedParams` would read the pair's origin for the `parent` param.

**ImpactSummary:** Seed-Child's recipe read gains the project case; tasks keep their own recipe field.
