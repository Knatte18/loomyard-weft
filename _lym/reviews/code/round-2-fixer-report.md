# Round 2 fixer-report

## Changes

- **F1 (BLOCKING), fixed as proposed**, commit `2f3ee7f7`.
  - `shedcheckkit.RecipeEngines(source []byte)` is new in `internal/testkit/shedcheckkit/recipeengines.go`; the kit's `doc.go` import line names `internal/shedbuild`.
  - `battenrecipe.RecipeEngines` is deleted from `internal/battenrecipe/names.go`, with its header mention and the unused `sort` import.
  - `internal/shedrecipe/coverage_guard_test.go` and `TestRecipeEngines_ReportsEveryEngineNameSortedOnce` call `shedcheckkit.RecipeEngines(recipes.BattenRecipe)`.
  - `fabricvocab.FailsBareCheck` is deleted and added as the unexported `failsBareCheck` in `internal/lymcwd/enforcement_test.go`, with its three calls renamed.
  - Both entries are dropped from `testOnlyCodeAllowed`.
- **F3 (LOW), fixed as proposed**, commit `43dc53a6`.
  - `internal/fabricengine/doc.go` opens the snapshot read-side paragraph with "Production code reads no snapshot", names `SnapshotWarpSHAForTest` as the one reader, and drops the snapshot-index sentence, the consumer-regenerates clause, and the `SnapshotWarpSHAForTest` mention in the cache-free-readers sentence.
- **Self-check follow-ups**, committed after the forks returned (see Self-check):
  - `coverage_guard_test.go` comments and failure messages say "engine set" instead of `RecipeEngines()`.
  - `fabricengine/doc.go` drops the staleness-check paragraph after the dangling `Warp-SHA` text and the polling-consumer advice.

- **Gate line-length repair**: the comment gate flagged four lines this round added.
  The doc comments of `TestRecipeEngines_ReportsEveryEngineNameSortedOnce`, `TestCoverageGuard` and `shedcheckkit.RecipeEngines` and one `doc.go` sentence pair are rewritten one whole sentence per line, with none over 119 characters.
  A second gate pass flagged the first split's mid-sentence breaks in the first two comments, which are now whole sentences.

Gate: `lym gate test` tier 0 over the touched packages (1464 passed, 0 failed) and tier 1 over `cmd/lym` (36 passed, 0 failed), both before the follow-ups; the follow-ups touch comments and two failure-message strings only.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `coverage_guard_test.go` still names `RecipeEngines()` per consumer; fixed as proposed.
- fix-proposal: LOW, `doc.go` staleness-check paragraph after the dangling `Warp-SHA` text still describes a consumer read; fixed as proposed.
- fix-proposal: LOW, `doc.go` polling-consumer advice; fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **F2 (MEDIUM)**: `_lym/landing/summary.md` is outside the fixer's write surface; the run's parent makes the edits the finding lists.
  Once F1 landed, the parent also removes `battenrecipe.RecipeEngines` and `fabricvocab.FailsBareCheck` from the allowlist list in the summary, and may add that `shedcheckkit` now holds `RecipeEngines`.

## Rulings

None.
