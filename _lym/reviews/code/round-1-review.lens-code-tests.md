```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke, or "none".

**Per behaviour, the pinning test**

1. `staleHub` start state (pair-a, pre-migration file sets, retired key, no sparse marker, no stamp):
   - `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` fails if the state drifts. It asserts one new commit per records worktree, the marker tracked after the walk, the retired key gone, and loom/shed contents.
   - `TestEnsure_CurrentStampLeavesConfigAlone` asserts `retiredKeyPresent` on the prime.
   - `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` breaks the config on the same start state (`ensure_integration_test.go:1055-1057`).
2. `newStaleHub(t)` returns a per-test copy of the template. Tests that mutate it in parallel (`TestEnsure_MidMergePairIsLeftAloneAndRetried`, `TestEnsure_PairDirectoryDeletedIsSkippedAsRemoved`) would collide if the copy were shared. `hubforge.requireIntact` and the fingerprint also fail a test that writes to the shared template (`template.go:353`).
3. `TestEnsure_HeldLockTimesOutWithoutStamping` at tier 0 (`stamp_test.go:85`):
   - The `*LockTimeoutError` type and `Path` assertion fails if `ensure` returns a different error or names a different path.
   - The no-stamp assertion goes through `readStamp(geom.StampPath())`. It fails if a stamp is written without the lock.
   - The timeout path is reached. With no stamp, `ensure` skips the early return (`walk.go:43`), creates the lock dir (`walk.go:52`) and calls `AcquireWriteLockWithin` (`walk.go:55`). It reads no git or hub before that, so an early git or hub error cannot mask the timeout.
   - It spawns nothing: no `gitkit` or `hubforge` import. It is parallel-safe on `t.TempDir` paths with `LockWait` at 50 ms.
   - The `EnsureOrWarn` "held lock is warned about" row pins the warning log. That is a different surface, so it does not duplicate this test.
4. Seam-copy ordering (`TestEnsureOrWarn_WarnsWithTheWayForward`, `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`, `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`):
   - No test pins it. It is a build-order discipline that `PATTERN-test-isolation` makes review discipline, not a scan. Not a finding.
   - The log and replay assertions are unchanged.
5. `arrangeUnpushedHubWide(t, h)`: each subtest still gets a fresh hub (`replayedHub`, `divergedHub`). Both subtests' assertions are unchanged, so the commit-count, stamp and divergence-log assertions still pin it.
6. `NewHub` to `CopyHub` at `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` and `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`: each keeps its assertions on the board commit, the stamp, the profiles content and the mail.
7. `testing.TB` signature changes on `seedRetiredKey`, `replaceOnce` and `commitPreMigrationFiles`: a compile-time change that adds no behaviour.

**Findings:** none.

**Tool calls:** none. The context already held every file this lens needed.
