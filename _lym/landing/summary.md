# Board README shows a task's profile in the Recipe column

The board's rendered tables showed a task's run profile as a `` `profile: <name>` `` bullet under the title in the Task column, while the Recipe column held only the recipe.
A profile is the lower level of a recipe, so the two now share one cell.

## What changes

- `recipeCell` in `internal/boardengine/render.go` renders the recipe, `tapestry` when the field is empty, followed by ` · ` and the profile when the task names one, such as `tapestry · light` or `darn · docs`.
  A task naming no profile takes its recipe's default profile at seeding, so its cell shows the recipe alone.
  The separator is the named constant `recipeProfileSeparator`.
- `entryCell` no longer adds the profile bullet; the brief and run-setting bullets are unchanged.
- The package doc in `internal/boardengine/doc.go` describes the new Recipe cell and drops the profile bullet.
- The render test's darn task with profile `docs` now expects `darn · docs` in the Recipe column and no profile bullet.

## Design decisions

- The cell shows the recipe alone whenever the task's `profile` field is empty, rather than resolving the hub's default profile from `profiles.yaml`.
  The renderer reads only the board, and the idea asks for the recipe alone exactly when the task takes the default.
