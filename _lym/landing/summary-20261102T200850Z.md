# Review default_recipe: resolve an empty recipe once, fix stale docs

Commit 8876e319, "A task with no recipe runs the board's default_recipe, darn by default", landed with no Code-Review.
This change reviews it and fixes what the review found.

## Verdict

The change is sound in design.
`default_recipe` is a cross-pair key in `board.yaml`, the template names `darn`, and Seed-Child, the README and `lym loom peers` all take a task's recipe from the board.
`ReadBoardType` now opens the board with `boardengine.OpenHub`, and an error there or from `GetTask` becomes a Stuck verdict whose reason names it, as before.
The tests pin the template default, a hub's own value, the default against a named recipe in the README cell and Seed-Child's recipe for a project, a named recipe and an empty one.

## Findings fixed

- **Two resolution points for an empty recipe.**
  `Board.RecipeOf` and the README's `recipeCell` each tested `task.Recipe != ""` and fell back to the default on their own.
  Both now call one unexported `Outputs.recipeOf`, so the README and Seed-Child cannot drift apart.
- **Stale package doc in `loomcli`.**
  The `lym loom peers` paragraph in `internal/loomcli/doc.go` still said an empty board recipe means tapestry; it now names the board's `default_recipe`.
- **`ReadBoardType`'s field doc in `battenshed/deps.go`** said the seam returns the task's own recipe field, then that the value is already resolved.
  It now says the value is the task's field or else the board's `default_recipe`.
- **The README stated the default twice**, as "`darn`, the default" in the recipe list and again in the sentence after it.
  The sentence naming `default_recipe` is now the one place.
- **`boardengine.Config.DefaultRecipe`'s comment** listed the recipe names and the template's value, which go stale apart from the field; both are dropped.
- **`TestLoadConfig_DefaultRecipe`** now calls `t.Parallel`, as the test-speed rule asks.
- **`TestRenderReadmeGolden`'s doc** said an empty recipe renders as tapestry; it now says the fixture's default recipe.

## Design decisions

- `lym loom peers` still writes the resolved recipe back into each task's `Recipe` field inside its board closure, and `peerRecipe` keeps its fallback that reads an empty name as tapestry.
  That fallback now serves only an empty self-run seed recipe, which `lym loom` reads as tapestry everywhere, so it is left as it is.
- A `default_recipe` value is still not checked at load.
  A bad name reaches the seeding site, which refuses it with a Stuck reason naming it, the same way it refuses a bad per-task recipe.

## Tests

The `TestLoadConfig_DefaultRecipe`, `TestRenderRecipeCellTakesTheDefaultRecipe`, `TestRenderReadmeGolden` and `TestBoardRecipe` tests pin the behaviour the refactor keeps.
No test was added: the refactor changes no behaviour.
