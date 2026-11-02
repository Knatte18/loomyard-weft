# Resolve an empty task recipe in one place and fix stale docs

The commit "A task with no recipe runs the board's default_recipe" landed without a review.
This change reviews it and fixes what the review found.
The behaviour is unchanged: a task with no `recipe` field runs the recipe `default_recipe` names in `board.yaml`, `darn` by default.

## What changes

- `internal/boardengine`: the empty-recipe rule had two copies, `Board.RecipeOf` and the Recipe-cell code in `render.go`.
  Both now call `Outputs.recipeOf` in `config.go`, the one place an empty recipe field is resolved.
  `recipeCell` takes the `Outputs` instead of a bare default string.
- Docs and comments no longer list recipe names where the setting is described, so the setting is stated once, at `Config.DefaultRecipe`:
  - the `template.yaml` comment and the `Config` and `Outputs` field comments;
  - the `README.md` recipe list, which no longer calls `darn` "the default" beside the paragraph that explains `default_recipe`.
- `internal/battenshed/deps.go`: the `ReadBoardType` comment says the value is the task's recipe or else the board's `default_recipe`.
- `internal/loomcli/doc.go`: the peer-recipe paragraph said an empty recipe means `tapestry`; it now says the board's `default_recipe`.
- `render_test.go`: the `readmeFixture` comment names the fixture's default recipe instead of `tapestry`.
- `config_test.go`: `TestLoadConfig_DefaultRecipe` and its subtests run in parallel.

## Review verdict

The rest of the commit holds: `battencli`, `boardcli` and `loomcli` carry no remaining literal fallback, tests pin both the default and the override, and the template documents the setting.
Nothing outside the commit's change needed a board note.

## To check by hand

- The `loomcli/doc.go` sentence about a peer's recipe describes behaviour this change does not touch; confirm it matches the code that seeds a peer's recipe from its board entry.
- This description was written from the diff alone; no tests were run while writing it.
