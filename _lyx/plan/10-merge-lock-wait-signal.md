# Card 10 — merge-lock-wait-signal

**Edit:**
- `internal/fabricengine#Fabric.MergeIn`
- `internal/fabricengine#Fabric.Merge`
- `internal/fabricengine#Fabric.MergeContinue`
- `internal/fabricengine#Fabric.MergeAbort`
- `internal/fabricengine_test#launchBlockedOnLock`

**Create:**
- `plan:internal/fabricengine#lockWaitHook` -> `var lockWaitHook atomic.Pointer[func(lockPath string)]`
- `plan:internal/fabricengine#acquireWeftWriteLock` -> `func acquireWeftWriteLock(lockDir string) (*lock.FileLock, error)`
- `plan:internal/fabricengine#lockWaitWaiters` -> `var lockWaitWaiters sync.Map`
- `plan:internal/fabricengine#installLockWaitDispatch` -> `var installLockWaitDispatch sync.Once`
- `plan:internal/fabricengine#AwaitWeftLockWaitForTest` -> `func AwaitWeftLockWaitForTest(t *testing.T, lockPath string) <-chan struct{}`
- `plan:internal/fabricengine#TestAcquireWeftWriteLock_SignalsOnlyItsOwnPath` -> `func TestAcquireWeftWriteLock_SignalsOnlyItsOwnPath(t *testing.T)`

**Uses:**
- `internal/fabricengine#weftWriteLockFile`
- `internal/fabricengine#WeftWriteLockPathForTest`
- `internal/fabricengine_test#TestMergeIn_ForeignStateAppearingWhileWaitingForLock_Refuses`

**Intent:** D7: `launchBlockedOnLock` assumes the verb is parked on the weft write lock 150 ms after it starts; where git spawns are slow the verb's pre-lock guard stage still runs when the fixture rewrites a tracked file or switches branch in the warp checkout, reads that as tracked dirt, and refuses `worktree dirty` instead of `ErrForeignMergeState`.
A test that waits for a fixed time stands in for a state it does not check, so the tests wait for the verb's own signal that it is about to wait for the lock.

- First, as a diagnostic that never lands: insert a 2-second sleep in `Fabric.MergeIn`'s pre-lock guard stage after the foreign-state check and before `pairDirtyReason`, which is where a slow guard sees the fixture's mid-wait tracked changes as dirt (a sleep before the foreign-state check would let the fixture's conflict finish first and refuse as foreign), run `lyx gate test --tags integration ./internal/fabricengine -- -run TestMergeIn_ForeignStateAppearingWhileWaitingForLock_Refuses -count=1` with the current helper, and expect the refusal to read `worktree dirty`; then remove the sleep.
  The commit body states the result; when the result is not `worktree dirty`, the body says the slow-guard cause is not confirmed and git 2.43 stays the open cause, and the fix below lands either way.
- `lockWaitHook` is an unexported test seam, nil in production and set only from `export_test.go`; an `atomic.Pointer` so parallel tests read it race-free.
- `acquireWeftWriteLock` joins `lockDir` with `weftWriteLockFile`, calls the hook with that path when one is installed, then returns `lock.AcquireWriteLock`'s result.
  `Fabric.MergeIn`, `Fabric.Merge`, `Fabric.MergeContinue` and `Fabric.MergeAbort` acquire the weft write lock through it in place of their direct `lock.AcquireWriteLock` call, keeping their own error wrapping; these are every verb `launchBlockedOnLock` launches, in `merge.go` and `mergelifecycle.go`.
  Bound: the hook observes and changes nothing, and no other lock site calls it.
- `export_test.go` holds `lockWaitWaiters`, a map from lock path to the channel a test waits on, and `AwaitWeftLockWaitForTest`, which installs, once through `installLockWaitDispatch`, a hook that takes the channel registered for its path out of the map with `LoadAndDelete` and closes it, registers a fresh channel for `lockPath`, and removes any entry still there on cleanup.
  Keyed by lock path, parallel tests each wait only for their own verb; a path with no registered channel, which every other merge test's verb reaches, and a second acquisition of a path already signalled are no-ops, so no channel is closed twice.
- `TestAcquireWeftWriteLock_SignalsOnlyItsOwnPath`, an untagged in-package test, registers through `AwaitWeftLockWaitForTest` for the lock paths of two temp lock dirs, calls `acquireWeftWriteLock` on the first, and asserts the first channel is closed and the second still open, then releases the lock.
  It then acquires and releases the first path again and a third, never-registered path, and asserts neither panics and the second channel is still open.
  A dispatch that ignored the path, closed twice or failed on an unregistered path fails it in the untagged tier, where the integration helper would only hang to its 30-second deadline.
- `launchBlockedOnLock` registers for `WeftWriteLockPathForTest(t, f)` before launching the verb, takes the lock, launches the verb, waits for the signal under a 30-second real-time deadline that only bounds a failing test (failing with the verb's result when the verb returns first), and then keeps its check that the verb has not finished, as a non-blocking receive on `done`.
  The fixed 150 ms wait is gone, and the file's comment names the signal.

**ImpactSummary:** Four merge verbs take the weft write lock through one helper with a nil-in-production observer; the lock's behaviour and every error message are unchanged, and only the integration helper waits on the new signal.
