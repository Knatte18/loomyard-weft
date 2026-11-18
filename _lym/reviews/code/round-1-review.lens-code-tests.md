```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that pins it.

**Per behaviour, the test that pins it**

1. `coalesceFlagged` passes the gate before it acquires and holds no lock while a pause holds it (`CoalescePushBothAt`, `PushPairAnchored`): `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`, which asserts the lock is free during the wait, one wait, and both sides pushed after the lift.
2. A detached child (`CoalescePushRequest`) defers at the gate before the lock: `TestCoalescePushRequest_CoveredCallDefers/CoveredCallDefersBeforeThePushLock`, which asserts the lock is free at exit, one exit, and both flags back.
3. `pushDeferral` puts back every owed flag, and its `pushed` parameter is removed: `TestPushDeferral_PutsBackEveryOwedFlag`. The rows it drops covered behaviour the diff removes, so no check was loosened.
4. `coalescePush` (`Bolt.Sync`, the board sync) waits at the gate before the board push lock: `TestSync_DeferrableExitsUnderACoveringPause/PlainCoveredWaitsThenReachesTheBoard`. With the board missing, a gate placed after the lock would error before any wait, so waits would be 0 instead of 1. The row pins the order.
5. `lockWeftForReplay` admitted before the weft write lock: none.
6. `Bolt.pullFirst` admitted before the board push lock: none.
7. `retryBoltPush` admitted before the board write lock: none.
8. `takeoverBranch.land` admitted before the records push lock: none.
9. `coalesceFlagged` releases its admission when an acquire misses or errors: none.
10. Pause skips a pair with no reed session, keeps its entry and interrupts the other pairs: `TestPause` row "a pair with no reed session is skipped and the pause goes on to the other pairs".
11. Status lists a pair and the prime with no reed session under `skipped` and still reads the other pairs: `TestStatus` subtest "a pair and a prime with no reed session are skipped and the other pairs still read".
12. Revive reports a pair with no reed session under `skipped`: none. `TestRevive` (`resume_test.go:569`) has no `sessionsErr` row.
13. Only `ErrNoSession` is skipped; any other `Sessions` error still fails pause, status and revive: none. Before the diff the fake never failed `Sessions`, and no row sets a different error.
14. Dobby starts no driver for a paused pair, a behaviour the diff relies on without changing it: `TestPlanPass` rows `VerdictPauseCovers{Probe,ChildWrite,Start,Lift,BattenWrite}GivesNone` (`dobbyengine/passes_test.go:65`), and `starts_test.go:648` "APauseCoveringTheFrontTaskHoldsItAndTheOthersBehindIt".

**Findings**

1. MEDIUM, design. Location: `internal/fabricengine/pull.go:503`, `bolt.go:301`, `pushclassified.go:811`, `takeoverwrite.go:173`.
   Four of the six gated sites have no test that would fail if their `pauserecord.Admit` were removed or moved after the lock: `lockWeftForReplay`, `Bolt.pullFirst`, `retryBoltPush` and `takeoverBranch.land`.
   Only the `coalesceFlagged` and `coalescePush` sites are pinned. This is one method gap, filed once.
   Evidence: the only `fabricengine` test files that install a gate are `pushdeferral_integration_test.go`, `teardownlock_test.go` and `export_test.go`, and none of them reaches those four functions.
2. LOW, design. Location: `internal/fabricengine/pendingflags.go:152-156`.
   No test checks that a missed or failed acquire releases the admission.
   A leaked admission leaves the process-wide `admissions` count above 0, so every later network call bypasses the pause (`pauserecord/hold.go:429`).
   Evidence: no `CoalescePushRequest` row runs with the lock held by another holder while a gate is installed; the diff's tests in `pushdeferral_integration_test.go` cover only a free lock.
3. LOW, design. Location: `internal/tenderengine/resume.go:479-483`.
   Revive's new skip of a pair with no reed session has no test.
   Evidence: `TestRevive` at `resume_test.go:569` has no row that sets `sessionsErr`; only `pause_test.go:496` and `resume_test.go:920` set it.
4. LOW, design. Location: `internal/tenderengine/pause.go:243-247` and `resume.go:691-696`.
   No row shows that a `Sessions` error other than `ErrNoSession` still fails pause, status or revive, so widening the match to every error would pass the suite.
   Evidence: `fakePair.sessionsErr` (`pause_test.go:33`) is only ever set to `errNoReedSession`.

Tool calls:
- grep of the tender, fabricengine and dobby tests for `sessionsErr`, `TestRevive`, gate installs and pause covers: the manifest did not show which tests reach the four gated sites, revive, or dobby's start path.
- read of `internal/dobbyengine/starts_test.go:630-680`: to confirm the covered-start row the manifest only located by grep.
