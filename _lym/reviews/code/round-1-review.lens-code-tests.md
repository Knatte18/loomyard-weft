```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff changes, name the test that pins it, or none.

**Per behaviour, the pinning test**

- **`Outputs.recipeOf` and `Board.RecipeOf`: an empty `Recipe` takes `DefaultRecipe`, a named one keeps its own.**
  - `TestBoardRecipe` (`internal/battencli/wire_test.go:480`) pins it through `boardRecipe`, which calls `Board.RecipeOf`, which calls `recipeOf`.
  - Row `TaskWithNoRecipeTakesTheBoardDefault` fails if the default branch breaks.
  - Rows `TaskWithDarnRecipe` and `TaskWithTapestryRecipe` fail if the override breaks.
  - The test uses `boardengine.New(Config{DefaultRecipe: darn})`, so the default is the one `Outputs` carries.
- **`recipeCell` now takes `Outputs` and resolves through `recipeOf`.**
  - `TestRenderRecipeCellTakesTheDefaultRecipe` (`render_test.go:899`) pins the empty-to-default and named cells with `DefaultRecipe: "darn"`.
  - `TestRenderReadmeGolden` (`render_test.go:864`) pins the same with `DefaultRecipe: "tapestry"`, plus the profile suffix, pipe escaping and the absence of the column in Notes and Done.
  - Both fail if either of the two call sites passes the wrong value or the signature change drops the default.
- **`TestLoadConfig_DefaultRecipe` now runs in parallel.**
  - It is still a test of the template default and the hub's own value, through `LoadConfig` and `Outputs()`.
  - `writeBoardConfig` uses `t.TempDir` only, with no `t.Setenv`, no `t.Chdir` and no shared fixture.
  - The `t.Parallel` calls are safe and match PATTERN-test-speed's "Tests run in parallel".
- **Doc and comment changes: `deps.go` `ReadBoardType`, `config.go` `DefaultRecipe`, `doc.go` peers paragraph, the README, and the `render_test.go` golden doc.**
  - None has a test, and none needs one. This is static prose or comment text.
- **Test economy.**
  - The diff adds no test, only `t.Parallel` calls and a doc line.
  - Nothing duplicates an existing test, and no review-fix test was added for a non-gap finding.
  - `TestRenderRecipeCellTakesTheDefaultRecipe` overlaps `TestRenderReadmeGolden`'s empty-recipe row. It dates from 8876e319, outside this diff, so I raise no redundancy finding.

**Findings**

None inside the diff's range.

**Outside the diff, for the handler to weigh**

- LOW, consistency, `internal/loomcli/peers.go:294-296`:
  - The peers board closure writes `board.RecipeOf(tasks[i])` back into `Task.Recipe`, and the changed `doc.go` paragraph now states that behaviour.
  - No test pins it. `peers_integration_test.go:50` names the recipe explicitly to avoid the default.
  - The behaviour comes from 8876e319, not from this diff, so raise it only if you count the new `doc.go` claim as in scope.
- The summary says an error from `OpenHub` or `GetTask` becomes a Stuck reason in `ReadBoardType`. The closure at `battencli/wire.go:1304-1317` has no test in my context, and `seamchild_test.go` only stubs the seam. This is unchanged by the diff, and the behaviour is the same as the summary describes.

**Tool calls:** none. My inherited context already held everything my procedure needed.
