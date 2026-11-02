---
round: 1
focus:
  - "internal/boardengine/config.go `Outputs.recipeOf`, board.go `Board.RecipeOf`, render.go `recipeCell`: does any other site in the repo still test a task's `Recipe` against \"\" and fall back to a literal or to `DefaultRecipe` itself (grep `\"tapestry\"`, `\"darn\"`, `.Recipe == \"\"`, `.Recipe != \"\"`, `DefaultRecipe`)? The decision record asks for one resolution point with no literal left elsewhere."
  - "internal/loomcli/doc.go `lym loom peers` paragraph now says an empty `recipe` field means the board's `default_recipe`: does the code in loomcli (the peer-recipe path, `peerRecipe`, and the closure that writes the resolved recipe back into `Task.Recipe`) agree, including for an empty self-run seed recipe, which the change description says still reads as tapestry?"
  - "Doc comments changed in battenshed/deps.go (`ReadBoardType`), boardengine/config.go (`Config.DefaultRecipe`, `Outputs.DefaultRecipe`, `recipeOf`), render.go (`recipeCell`) and render_test.go: does each state what the code does, and is the `default_recipe` setting stated once in the docs (README), with no second statement left in README or other docs the diff did not touch?"
  - "Change description `_lym/landing/summary.md`: check each claim against the diff, in particular that `TestRenderRecipeCellTakesTheDefaultRecipe` and `TestBoardRecipe` exist and pin the default and the override, that `ReadBoardType` opens the board with `boardengine.OpenHub` and turns an open or `GetTask` error into a Stuck reason, and the claim that no behavior changed."
  - "Test economy and coverage: the diff changes the `recipeCell` signature and adds `Outputs.recipeOf`; does an existing test cover the default and the named-recipe branch through the public surface, so no uncovered behavior and no duplicate test is added? Does `t.Parallel` in `TestLoadConfig_DefaultRecipe` hold with `writeBoardConfig` and the test-speed rule in the repository's instructions?"
  - "Repository rules: read the constraints document and the PATTERN entries the change touches (value vs pointer receiver on `Outputs`, unexported helper placement in config.go, comment-convention rules for doc comments); is anything the diff changes in conflict with them?"
---

The review range holds one commit, 9d996439, which is the fix for the findings on 8876e319.
Judge that commit's diff as the fix, and check the code it touches in 8876e319 only where the diff's claim depends on it.
