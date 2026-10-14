## Fixer-report, round 1

### Changes

- F1 fixed with an adjustment: `spawnPairPush` in `internal/fabriccli/spawn.go` is now `fabricengine.SpawnPairPush` (signature `func(*lyxcwd.Location) error`), `SetPairPushForTest` and the `TestMain` replacement take the new signature, and the header comment is updated.
  The review's fix calls `spawnPairPush(l)` with the verb's own location.
  In `runAdd`, `l` is the hub-side location, not the new pair's, so `runAdd` resolves the pair's location with `lyxcwd.ResolveWorktree(codeWorktree)` first, as batten's task locator does.
  A resolve failure is reported as `push_error`, like a failed spawn.
  The `push_spawned` entries keep their `WorktreePath` and `RecordsWorktreePath` targets.
  `SetPairPushForTest` also returns the previous seam, so a serial test can restore it.
- F2 fixed: a subtest `AddSpawnsThePairPushAndReportsAFailedSpawnWithoutFailing` in `TestRunCLI_WeftSkipPushScenario`, with a success row and a failing-spawn row.
  The success row asserts one spawn at the new pair, two `push_spawned` entries and no `push_error`.
  The failing row asserts `push_error` carries the error, the exit is ok, no `push_spawned` entry exists and the pair's worktree remains.
  It lives in that scenario rather than the hub-mutation one because the seam is process-global and only the former is serial.
  `TestSpawnPairPush_HandsTheDetachedPushBothSidesOfThePair` in `internal/fabricengine/spawn_test.go` asserts the paths handed to `spawnDetachedPushFn` and the returned error.
  `fabricengine` stays within its test-budget row, so the budget file is unchanged.
- F3 fixed, differently from the proposal: the create-row test cannot swap `spawnPairPush`, because it is parallel and the seam is process-global.
  The integration `TestMain` seam now counts pushes per worktree path (`pairPushCounts`), and `TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating` asserts one push for the new pair after the create row and the re-entered row.
  The `PushSeed` error row already exists as `TestSeedChild_FailedPushWarnsAndStillReturnsDone` in `internal/battenshed/seamchild_test.go`, so none was added.
- F4 fixed as proposed in `internal/fabricengine/destroy.go`.

### Disputed

- F5, premise false: `rollbackAdd`'s `warpTok` is the `createdToken` that `createGitWorktree` returns for the worktree, while `warpPush.tok` is the `createdBranchToken` it returns for the branch (`add.go:184`, `add.go:196`).
  They are different values and types, so the wrapper cannot be replaced by the existing parameter, and the review's own fix says to skip in this case.
  The `addWarpBranch` wrapper is untouched.

### Deferred

None.

### Verification

`lyx gate test` and `lyx gate test --tags integration` are green for `fabricengine`, `fabriccli`, `battencli` and `battenshed`, the integration tier for the first three.
