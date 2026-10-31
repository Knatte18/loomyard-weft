```yaml
lens: code-affected-set
outcome: reported
```

Task restated: the code-affected-set lens, meaning what the rename forces outside the lines the diff touches.

## Affected sets, per change

- **Recipe value `RecipeTapestry`, `RecipeNames`**: callers, tests, docs.
  - The Go callers and tests I saw were updated: shedcli, loomcli, battencli, loomengine, loomrecipe and envkit.
  - Covered, except the stale literals in F2–F5 below.
- **`Loom-Preflight` row, `tapestryPreflightEntry`, constants, file move**:
  - The registry, the recipe yaml, the loomshed file move and the registry test list are covered.
  - **Not covered**: the retired-row table (F1).
- **Empty recipe means `tapestry`**: boardengine `defaultRecipe`, `recipeCell`, the doc comments and the `task.go` refusal text are covered. `docs/overview.md` and `README.md` are not (F2).
- **Recipe-name values in docs, fixtures and sandbox suites**: `--recipe` and compact `"recipe":"loom"` forms are covered. The spaced `{"recipe": "loom"` form and the prose lists are not (F3, F4).
- **Prose "loom" for the variant**: the phrase table covered the listed phrases. Stale spots remain in F2, F5 and F6.
- **PATTERN.md and `pattern/`**: the changed entries (`PATTERN.md`, gate-slot, leaf-packages) show no leftover. The directive declares no entries to land, and no `Pattern-Departure:` trailers appear in the range's commit messages.

## Findings

**F1 — BLOCKING, design.** `internal/loomrecipe/loomrecipe.go:18`
- `TapestryRetiredRows()` returns only `{"Merge-In": loomshed.NameParentAlign}`.
- The decision record's Constraints require that the `Loom-Preflight` rename "goes with a retired-row entry", because row names are durable resume identities.
- The focus directive says the same.
- The code has no `"Loom-Preflight": "Tapestry-Preflight"` entry.
  - A run parked on `Loom-Preflight` fails to resume.
  - `peers.go:88` and `arm.go:197` consume the table.
- `TestRenameRetiredRows` (`internal/loomcli/wiring_darn_test.go:292`) pins only `Merge-In`.
- The recipe file comment (`tapestry-recipe.yaml:58`) and `loomrecipe/doc.go:27` describe the table without the new entry.
- Fix: add the entry, add a test row for it, and update the doc comment.

**F2 — MEDIUM, scope.** `README.md:58`, `docs/overview.md:650`
- `README.md:58` still reads "`loom`, the default, for a task big enough to need a discussion and a plan".
- `docs/overview.md:650` reads "`recipe` (`loom`, or `darn` ..., empty meaning `loom`)".
- Both state the old recipe vocabulary and the empty-recipe default, which D4 changed.
- Fix: say `tapestry` in both places.

**F3 — MEDIUM, scope.** `tools/sandbox/SANDBOX-CORE-SUITE.md:250`
- It instructs the operator to write the seed `{"recipe": "loom", "driver": "go"}`, with a space after the colon.
- The tables' compact `"recipe":"loom"` key does not match that form.
- No alias exists now, so a sandbox run following this line gets an unknown-recipe refusal.
- Fix: change the value to `tapestry`.

**F4 — LOW, consistency.** `internal/battencli/arm.go:203`
- The comment says "a prime seed carrying "loom" would make ... arm loomcli".
- It names the old recipe value.
- Fix: write `"tapestry"`.

**F5 — LOW, consistency.** `contracts/specs/shed-recipe-spec.md:113`
- It says "the coverage guard driving loom's real row list".
- This is the variant's own row list, so "loom's" should read "tapestry's".
- The commit message of `b44cdabf` says such spec lines were fixed.
- `shed-recipe-spec.md:112` ("loom-specific constructors") is the same case.

**F6 — LOW, consistency.**
- These still use `loom` as a recipe name:
  - `internal/boardengine/board_test.go:249` asserts `Recipe != "loom"`.
  - `internal/boardengine/task_test.go:187` uses `"recipe": "loom"` for a project.
  - `internal/runprofile/runprofile_test.go:56-61`, `internal/configreg/runsettings_test.go:475` and `internal/configcli/configcli_test.go:1118` use `recipe "loom"` as a profile recipe block or default key.
  - Map keys and case names such as `"loom": recipes.TapestryRecipe` appear in `loomrecipe/recipe_test.go:99`, `decisionswitches_test.go:26`, `fansettings_test.go:109`, `envkit_test.go:76` and `loomcli/wiring_darn_test.go:240`.
- Some of these tests pass only because the boardengine and runprofile layers do not validate the vocabulary.
- They keep the retired name alive as test data for the variant.
- Fix: change the values to `tapestry`, except where `loom` is the config module key.

## Tool calls
1. Read `internal/loomrecipe/loomrecipe.go` (lines 10–40) and grepped contracts, plugins, templates, `README.md` and PATTERN docs for recipe-value `loom`. My context lacked the retired-rows body and the docs state.
2. Grepped the whole tree for `"recipe": "loom"`, `recipe loom` and similar, and read `board_test.go:245-252` and `battencli/arm.go:198-208`. My context lacked these spots.
