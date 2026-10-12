# Fixer report — plan round 5

## Changes

- **F1** (fixed as proposed): card 15's **Edit:** adds `internal/fabriccli_test#TestRunCLI_CloneEndToEnd` and `internal/fabriccli_test#TestRunCLI_AnchoredHubScenario`.
  Card 15's Intent states the new expectation of `CleanHubScenario`'s `CloneConfigCommit_PairInheritsConfigs` step (loom resolves from the board's repo default through `loomengine.LoadConfig` over `fabricengine.ConfigDirs`), of `TestRunCLI_CloneEndToEnd` and of `AnchoredHubScenario`'s `CloneConfigCommit_AnchorScoped` step, and names the search that found them so a later round can rerun it.
  Card 16's `TestRunCLI_CleanHubScenario` sentence adds that the `PairInheritsConfigs` step asserts the pair inherits the sparse marker.
  I reran the search over every test that reads `configengine.ConfigFile`/`ConfigFileRel` and uses `hubforge` or `CloneAndWire`.
  The remaining hits read hub-wide modules, seed or commit the file first, or tolerate its absence, so the three tests are the complete set.
- **F2** (fixed as proposed): card 15's reseed case now deletes loom's repo default, never `batcher`'s.
  Card 16's line 69 states the case's post-migration outcome: loom's default is reseeded from the full prime file, the prime's and the pair's `loom.yaml` are stripped empty and deleted, and each deletion is committed in that worktree's one reconcile commit.
- **F3** (fixed as proposed): card 15's **Edit:** replaces `internal/hubreconcile/ensure_integration_test.go` with `internal/hubreconcile_test#TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` and `internal/hubreconcile_test#runStaleBuildReconcile`.
  Card 16's **Edit:** adds `internal/hubreconcile_test#runStaleBuildReconcile`.

## Disputed

None.

## Deferred

None.
