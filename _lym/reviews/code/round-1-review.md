---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/boardengine/template.yaml:3
    summary: "The template comment on default_recipe still lists the recipe names that the same fix dropped from the Config.DefaultRecipe doc as stale-prone"
    fix: "In internal/boardengine/template.yaml line 3, shorten the trailing comment to 'recipe a task with no recipe field runs' and drop the trailing list 'darn, tapestry or sampler', so the template no longer enumerates recipe names apart from the value it sets."
    origin: lens:code-affected-set
---

### [LOW] Template comment still enumerates recipe names

**Location:** internal/boardengine/template.yaml:3
**Issue:** Commit 9d996439 removed "tapestry, darn or sampler" and "The template names darn" from the `Config.DefaultRecipe` doc in `internal/boardengine/config.go`, because a list of names goes stale apart from the field.
The template's own comment on the same key still reads "recipe a task with no recipe field runs: darn, tapestry or sampler".
It is the user-facing copy of the enumeration the fix removed, so the two now disagree on whether the recipe set belongs next to the key.

## Rejected

- code-affected-set finding 2 (`docs/overview.md:364`, `docs/overview.md:680`, `plugins/ly/skills/board/SKILL.md:52` also state the default): each site describes the setting for a different reader, as a board.yaml key, as the batten seeding rule and as an operator's skill instruction.
  The duplicate the decision record targets, "`darn`, the default" and then the `default_recipe` sentence in one README passage, is the one the commit removed, and README.md:61 is now the only README statement.
- code-tests note on `internal/loomcli/peers.go:294-296` (the write-back of the resolved recipe is unpinned by a test): the behaviour comes from 8876e319 and this diff adds only a `doc.go` sentence describing it, so no behavior the diff adds lacks coverage.
- code-tests note on `TestRenderRecipeCellTakesTheDefaultRecipe` overlapping `TestRenderReadmeGolden`: the test dates from 8876e319, outside the review range, and the diff adds no test.

## Verdict notes

- One resolution point: `Outputs.recipeOf` in `internal/boardengine/config.go` is the only place a task's empty `Recipe` is resolved; `Board.RecipeOf` and `recipeCell` call it, and the grep for `"tapestry"`, `"darn"`, `.Recipe ==`, `.Recipe !=` and `DefaultRecipe` finds no other fallback in non-test code.
  The `peerRecipe` fallback to tapestry in `internal/loomcli/peers.go` serves only an empty self-run seed recipe, as the change description says, and the `doc.go` peers paragraph agrees with the board closure that writes the resolved recipe back.
- The change description's claims about `TestRenderRecipeCellTakesTheDefaultRecipe`, `TestBoardRecipe` and `ReadBoardType` hold against the diff and the code read; the commit adds no behavior change and no test.
- `t.Parallel` on `TestLoadConfig_DefaultRecipe` follows PATTERN-test-speed: `writeBoardConfig` uses `t.TempDir` and no process-global state.
