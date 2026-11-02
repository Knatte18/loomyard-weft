# A task with no recipe runs the board's default_recipe, darn by default

A task whose `recipe` field is empty used to run tapestry, hard-coded in two places that each decided it alone: the README's Recipe cell and the Seed-Child producer.
A small task promoted without a recipe therefore started a full tapestry run.
The default is now a hub setting, `default_recipe` in `board.yaml`, and the template names `darn`.
A hub that wants the old behaviour sets `default_recipe: tapestry`.

## What changed

- `boardengine.Config` reads `default_recipe` (hub-wide, cross-pair like the other board keys) and carries it into `Outputs`.
  `Board.RecipeOf` returns a task's own recipe, else the configured default.
- The README's Recipe cell takes the default from `Outputs` instead of the removed `defaultRecipe` constant.
- Seed-Child's `ReadBoardType` seam opens the hub's board with its config (`boardengine.OpenHub`) and resolves the recipe through `Board.RecipeOf`.
  `battenshed` loses its `defaultChildRecipe` fallback, since the seam never returns an empty recipe.
- `lym loom peers` resolves each peer's recipe through the same `RecipeOf`, so a peer with no recipe reports darn's rows.
- The README, `docs/overview.md`, the board skill, `lym board upsert`'s help and the `boardengine` package doc no longer say an empty recipe means tapestry.

## Design decisions

- No fallback constant stands in for a missing `default_recipe`: a `Config` built without one, as tests do, renders and resolves an empty recipe.
  The template always supplies the key for a loaded config, and a hard-coded fallback would reintroduce the choice this change moves into config.
- The value is not validated at load.
  The seeding site already refuses an unknown or unsupported recipe name with a Stuck reason naming it, which covers a bad `default_recipe` the same way as a bad per-task recipe.
- `lym loom` keeps reading an empty recipe name as tapestry for a run with no seed.
  That is a standalone run's own seed default, separate from a board task's recipe, and no board entry reaches it.

## Tasks that change recipe

No task on the board has an empty recipe.
The five open tasks, `operator-verbs-readable-output`, `default-recipe-darn`, `plan-split-replay`, `finalize-fix-outside-lock` and `gitexec-remote-bounds-load-flaky`, name `darn` explicitly, and every other open entry is a note, which runs nothing until promoted.
A note promoted without a recipe now runs darn, so the operator sets `tapestry` on any promotion that needs it.

## Deploy

An existing hub's `board.yaml` lacks `default_recipe`; the template supplies `darn` until `lym config reconcile --apply` writes the key.

## Tests

- `TestLoadConfig_DefaultRecipe` pins the template's darn when the key is absent and a hub's own value when set.
- `TestRenderRecipeCellTakesTheDefaultRecipe` pins the cell for an empty and a named recipe; the README goldens run with `tapestry` as the hub's default.
- `TestBoardRecipe` pins Seed-Child's recipe for a project, a named recipe and an empty one.
