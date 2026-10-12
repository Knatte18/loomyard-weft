# Round 4 fixer report — plan

## Changes

- **F1** (fixed as proposed): card 15 adds `internal/hubforge#TestNewHub` and `internal/orchcli#TestOrchIntegration_Refusals` to **Edit:**.
  Its Intent now states the new expectations of `TestRunCLI_CleanHubScenario`'s `CloneConfigCommit_WeftPrimeCleanAfterClone` and `CloneConfigCommit_OneCommitNotOnePerModule` steps, `TestNewHub`'s `ConfigMaterializedWithoutSeeding` and `SeedConfigRedundantSeedDoesNotFatal` steps, and the `hub config reconcile` subtest of `TestOrchIntegration_Refusals`.
  Premise checked in `internal/hubforge/hub_test.go`, `internal/fabriccli/cleanhub_integration_test.go` and `internal/orchcli/refusal_integration_test.go`: each reads or counts the clone's old prime-side output.
- **F2** (fixed as proposed): card 16 adds `internal/hubreconcile_test#seedRetiredKey` to **Edit:**.
  Its Intent says `newStaleHub` writes the full per-worktree file set first and calls `seedRetiredKey` after it, and that `seedRetiredKey` commits `batcher.yaml` as the template with the retired key in place of `orientation` plus the one differing key, so the file survives the strip non-empty.
- **F3** (fixed as proposed): card 16's Intent says the hook in `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` keeps `markMergeRecord`'s clear func, which the test calls after the restore assertions before running `Ensure` again and asserting the pair is stripped and committed with the marker.
  The migration-assertions sentence now says the next walk strips it once the merge record is cleared.
- **F4** (fixed as proposed): card 16's Intent says `runStaleBuildReconcile` removes the stamp at `geom.StampPath()` before its second `Ensure`, then asserts no new prime, pair or board commit and byte-identical per-worktree files and markers.

## Disputed

None.

## Deferred

None.
