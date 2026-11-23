```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I traced what the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered:**

1. **Rename `TestPRFindingsCLI_SendsBackOnATapestryRun` → `TestPRFindingsCLI_SendsBackBlockingFindings`.**
   - A repo-wide grep outside `.lym`, `_lym` and `.scratch` finds the old name nowhere: no doc, no benchmark report and no comment still uses it.
   - The `_lym/discussion/decision-record.md` "Added after Discussion" entry names the old name, but it is the answer key and not a subject.
   - The new name's doc comment opens with the new name.
   - Covered.
2. **New helper `blockingFindingsRun`.**
   - It is a test-only function in a `_test.go` file, so `PATTERN-test-only-code` holds.
   - The hub fixture comes through `hubforge.CopyHub`, so `PATTERN-test-isolation` holds.
   - Its one caller is the subtest at `prfindings_integration_test.go:63`.
   - Covered.
3. **The test now writes `seed.json` through `shedrun.WriteSeed` with the recipe and `DriverGo`.**
   - Readers: `resolveRecipe` (`internal/loomcli/arm.go:110`), then `c.recipe`, `prReviewSendBackEdge` and `loomrecipe.EngineRow`.
   - `WriteSeed` validates the recipe through `ValidateRecipe`, and both `RecipeTapestry` and `RecipeDarn` are in `RecipeNames`.
   - `contracts/recipes/darn-recipe.yaml:285-296` has a PR-Review row with `on_reroute: Code-Bouncer` and a Finalize row, which matches the new doc comment's claim that darn's first landing row is Finalize.
   - Covered.
4. **Doc-comment and file-header claims.**
   - The file header (lines 1-5) still describes the file correctly: it wires the landing seams before building the shed.
   - The test doc comment's per-recipe claims match `darn-recipe.yaml` and `prReviewSendBackEdge`'s comment (`prfindings.go:285-289`).
   - Covered.
5. **Repository rules.**
   - `PATTERN-test-speed`: `tiergate.Require(t, 1)` is the first call, `t.Parallel` is in the test and in each subtest, and many items of one kind are covered by one table-driven test. Covered.
   - Declared PATTERN changes: none declared, so nothing to land.
   - Stencil wording: no stencil changed.
6. **Commit messages and departures (commit 6812beb1e).**
   - No `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed because no assertion was widened: the exit-code and `send_back` checks are unchanged.
   - The commit has no `Root-Cause:` trailer. The diff makes no fix, and the amended entry's second added entry records the bug as already fixed on the base by 6fa7f1740. The bullet's trigger, "asks to fix a bug" with a fix in the diff, does not apply, so this is not a finding.

**Findings:** none.

Outside my lens, noted only: the tapestry row now seeds an explicit `tapestry` recipe, where the base test ran seedless and relied on `resolveSeedRecipe`'s default. The seedless default path of this verb is no longer pinned by this test. Whether another test pins it is for the code-tests lens.

**Tool calls:**
1. Repo-wide grep for the old and new test names and the helper name, to find stale references (docs, reports, comments) the manifest did not cover.
