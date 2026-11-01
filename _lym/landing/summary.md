# Board README shows a task's profile in the Recipe column

The rendered board README showed a task's run profile as a `profile: <name>` bullet under its title in the Task column, while the Recipe column held only the recipe.
The profile is the lower level of the recipe, so it now sits in the same column: `tapestry · light`, or the recipe alone when the task names no profile and takes its recipe's default.
The profile bullet under the title is gone; run-setting bullets are unchanged.

## Design decisions

- `recipeCell` in `internal/boardengine/render.go` appends the separator `recipeProfileSeparator` (` · `) and the profile to the recipe, which is still `tapestry` when the recipe field is empty.
  A profile on a task with no recipe therefore renders as `tapestry · <profile>`.
- The profile goes through `cellText`, so a pipe in it is escaped and the entry stays one table row.
- `entryCell` no longer reads the profile. The Notes and Done tables have no Recipe column and show no profile.
- The `doc.go` layout description and the `render_test.go` golden are updated to match. The fixture gains a profile with a pipe on a task with no recipe, beside the existing recipe with a profile.

## To check by hand

Nothing was left open. Regenerate a README from a real board to see the column on live tasks if wanted.
