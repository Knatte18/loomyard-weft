# Card 13 — A standing move record stops Add, reconcile, pull, the fast-forward, push, checkout and the origin reconcile

**Edit:**
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#Fabric.pullWeftArm`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#Fabric.fastForwardWeft`
- `internal/fabricengine_test#TestFastForwardPair_MovesBehindPairToUpstream`
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#BranchOutcome`
- `internal/fabricengine#BranchOutcome.stopWayForward`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine_test#TestCheckout_RefusesAnotherAnchor`
- `internal/fabricengine_test#TestAdd_FromParentPair`
- `internal/fabricengine_test#TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot`
- `internal/fabricengine_test#TestPull_WeftDivergedWarpAdvancesCleanly`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `cmd/lyx#destructiveGuardAllowlist`

**Create:**
- `internal/fabricengine#MoveRecord`
- `internal/fabricengine#MoveRecordPath`
- `internal/fabricengine#ReadMoveRecord`
- `internal/fabricengine#RemoveMoveRecord`
- `internal/fabricengine#MoveInProgress`
- `internal/fabricengine#moveLockPath`

**Uses:**
- `internal/fabricengine#BranchAnchorStopped`
- `internal/fabricengine#ErrAnchorChange`
- `internal/fabricengine#AnchorStopError`
- `internal/fabricengine#weftLockDirName`
- `internal/fabricengine#ReconcilePairResult`

**Intent:**
D7 step 1's record and lock, and the refusals a standing record raises; card 15 writes the record.
`MoveRecord` holds `A`, `B`, `Kind` (`move` or `integrate`), `U` (the integration's upstream tip, empty until step 4), `PID` and `StartTime`, JSON through `internal/state` (`PATTERN-tolerant-records`).
`MoveRecordPath` is `<weftPath>/.weft/anchor-move.json` and `moveLockPath` `<weftPath>/.weft/anchor-move.lock`, both in the weft lock directory, which git excludes and no move touches.
`ReadMoveRecord` reports an absent record as not found.
`RemoveMoveRecord` is the one remover of the record, called by card 15's `Apply` failure path, `Finish` and `Close`, and by card 18's create: a tolerant `os.Remove` of `MoveRecordPath`, an absent record no error, in the new file `internal/fabricengine/moverecord.go`, since `internal/state` has no delete primitive, as `ClearPushFailure` removes the failed-push record.
`destructiveGuardAllowlist` gains an entry for `internal/fabricengine/moverecord.go` whose reason is that its `os.Remove` deletes fabric's own move record in the weft's lock directory, fabric-internal metadata, never operator content.
`MoveInProgress` reads the record and try-acquires the move lock without blocking, releasing it at once: with no record it reports not standing; with the lock held it names a change in progress, the record's pid, and waiting for it, then `lyx fabric anchor <slug> --set <B>`; with the lock free it names `lyx fabric anchor <slug> --set <B>`, the re-run that recovers.
`slug` is the worktree's name as `lyx fabric anchor` takes it, the prime's included.

Callers that act on a worktree with a standing record, each before touching anything:
`Topology.add` refuses a fork point whose weft has one (D5), with `MoveInProgress`'s text;
`Topology.Reconcile` reports such a pair failed with that text in `ReconcilePairResult.Error`, its wiring untouched, and goes on (D8);
`Fabric.pullWeftArm` returns a `*WeftStopError` wrapping `ErrAnchorChange` with that text as `WayForward`, and `Fabric.Pull` returns it before the warp arm as card 9 does for the anchor stop;
`Fabric.fastForwardWeft` skips the weft pull with its existing warning, naming that text, and the code side goes on, as card 9 does for the anchor stop;
`retryWeftPush`, when the upstream has commits the weft lacks and so the push must replay, returns an `*AnchorStopError` with that text, which `classifyFailure` records as `anchor_change` (card 9); a fast-forward push is not stopped;
`Topology.Checkout` refuses a worktree whose weft has one, before anything is switched, since a re-run recovers against the branch the record was written on;
`originMover.move` leaves the weft unmoved as `BranchAnchorStopped`, `BranchOutcome` gaining `MoveWayForward string`, which `stopWayForward` returns when set.

Tests write a record by hand at `MoveRecordPath` (and hold the lock with `lock.TryAcquireWriteLock` for the held rows):
`TestAdd_FromParentPair`'s refusal table gains `MoveRecordOnForkPoint`, lock free and lock held, each naming `lyx fabric anchor beam --set <B>` and creating nothing;
`TestReconcile_RefusesEmptyAnchorMarkerInsteadOfWiringAtTheRoot` gains a row: a pair with a record is failed with that way forward, its junctions unchanged, other pairs reconciled;
`TestPull_WeftDivergedWarpAdvancesCleanly` and `TestPushFailureClasses` gain a row each: a record on the weft stops the pull, and the replaying push, with both sides unchanged;
`TestFastForwardPair_MovesBehindPairToUpstream` gains a row: a record on the weft leaves the weft `HEAD` unchanged while the code side still lands on its upstream tip;
`TestReconcileOrigin_Moves` gains a row: a record on a pair's weft leaves it unmoved as `anchor_stopped` with the move's way forward, and no error;
`TestCheckout_RefusesAnotherAnchor` gains a row: a record on the worktree's weft refuses a checkout to a branch at the same anchor, naming `lyx fabric anchor`, with both sides still on their branches.

**ImpactSummary:** seven weft writers, checkout and the fast-forward's weft pull included, gain a pre-move check on one new record file; with no record on disk every path behaves as today, one outcome field is added, and the destructive guard's allowlist gains the record remover's file.
