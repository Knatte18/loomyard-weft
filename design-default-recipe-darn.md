# A task with no recipe runs darn, not tapestry

`default-recipe-darn` · Task · enhancement, board, batten · running · Darn-Write

## Seen (orch, 2026-11-01)

A task with no recipe runs tapestry: `boardengine` reads an empty `recipe` as tapestry (`defaultRecipe` in `render.go`, the `Recipe` field comment in `task.go`), and the seeding site resolves it the same way.
A note promoted without a recipe, `plan-spec-split`, a rewrite of one markdown file, so started as a full tapestry run before the operator stopped it.
Darn is what most tasks need; tapestry is the exception for large or design-heavy work.

## Idea (operator, 2026-11-01)

- The default recipe becomes darn: a task with no recipe runs darn, and the README's Recipe column shows darn for it.
- The default is a hub setting in `board.yaml` (for example `default_recipe`), with darn as the template's value, so a hub can choose.
- The board skill, the CLI help and the docs that say "empty means tapestry" follow.
- Tasks already on the board with an empty recipe: the change names them, so the operator can set tapestry on any that need it before it lands.
