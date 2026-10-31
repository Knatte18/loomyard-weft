```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing per behaviour the test that pins it or none.

**Per behaviour**

1. `shedrun.RecipeTapestry` is `"tapestry"` and the vocabulary is sorted. Pinned by `TestRecipeNames` (`internal/shedrun/seed_test.go:186`) and `TestRecipes_KeySet` (`internal/shedcli/table_test.go:31`).
2. `ValidateRecipe` accepts `tapestry`. Pinned by the `tapestry` row of `TestValidateRecipe` (`internal/shedrun/seed_test.go:167`).
3. D6: the old name `loom` is refused as an unknown recipe, and the refusal lists `tapestry`. **None.** `TestValidateRecipe` has refusal rows only for `beam` and `someday`, and no row asserts that the message lists `tapestry`. The `unknown recipe` rows in `internal/shedcli/cli_test.go:174,200` were not checked for their recipe value or for the listed names.
4. An empty board `recipe` shows `tapestry` in the README Recipe cell. Pinned by the `tapestry` cells in `internal/boardengine/render_test.go`.
5. The `ProjectRecipe` refusal text says `tapestry or darn`. I found no test that asserts this text (none among the `render_test.go` rows I read). **None, found.**
6. Seed-Child seeds `tapestry` for an empty type. Pinned by `internal/battenshed/seamchild_test.go:365`, the `AbsentSettingsRemoveEveryLocalFile` row.
7. `TestRenameRetiredRows` (`internal/loomcli/wiring_darn_test.go:292`) pins the `Merge-In` rename for the tapestry table. No test pins `Loom-Preflight` → `Tapestry-Preflight`. See F1.
8. The recipe file and registry row `TapestryPreflight`: pinned by the registry list in `internal/shedrecipe/registry_test.go` and the coverage guard in `internal/loomrecipe`.
9. The `tools/tapestryrename` tables pin one case each in `TestSubstitute`:
   - a listed token is substituted;
   - an unlisted `loom` is left alone;
   - an excluded path is skipped (`TestRenameTree`, `docs/research`).
   `TestRenameTree` also covers moves and idempotence.
   - The `moves` entries for `internal/loomshed/loompreflight*.go` are untested. `TestRenameTree` exercises only `moves[0]`.
   - Most `valueSubstitutions` fragments, about 40, have no row. They are one-off test-fixture rewrites, whose result the later gate runs check.

**Findings**

- **F1 — BLOCKING, design.** `loomrecipe.TapestryRetiredRows` (`internal/loomrecipe/loomrecipe.go:18`) returns only `{"Merge-In": ParentAlign}`. The Constraints require the `Loom-Preflight` rename to ship with a retired-row entry, because row names are durable resume identities. The focus names it as the one entry that must stay. The entry is missing, and so is any test that a status parked on `Loom-Preflight` resumes on `Tapestry-Preflight`. `TestRenameRetiredRows` (`internal/loomcli/wiring_darn_test.go:292`) has no row for it. The fix is to add `"Loom-Preflight": loomshed.NameTapestryPreflight` (or the constant's actual name) to the map, plus a `TestRenameRetiredRows` row.
- **F2 — MEDIUM, scope.** No test pins D6: `loom` is refused as an unknown recipe and the refusal lists `tapestry`. The fix is to add a `"former loom name", "loom", true` row to `TestValidateRecipe` (`internal/shedrun/seed_test.go:160`). For the listing, add an assertion on the `shedcli` unknown-recipe refusal (`internal/shedcli/cli_test.go:174`) using `--recipe loom`.
- **F3 — LOW, scope.** The `moves` entries for `internal/loomshed/loompreflight.go` and `loompreflight_test.go` are never exercised, because `TestRenameTree` runs only `moves[0]`. If those entries were wrong, the tests would not fail. The fix is to give `TestRenameTree` a second moved file, or to assert `len(moves)` against the tree.

**Tool calls**
1. A `sed` of `internal/loomrecipe/loomrecipe.go` plus greps for `Loom-Preflight`, `unknown recipe` and the render and seed tests. My context did not show `TapestryRetiredRows`' body or the tests' tapestry rows.
2. A `sed` of `internal/shedrun/seed_test.go:160-182` and greps for the Preflight tests, to confirm the `TestValidateRecipe` rows and the missing retired-row test.
