# Card 17 — origin-reconcile-ended

**Create:**
- `plan:internal/fabricengine#reconcileEnded` -> `func reconcileEnded(l *lyxcwd.Location, scope ReconcileScope, seams OriginSeams, ended EndedSet, res *OriginReport) error`
- `plan:internal/fabricengine_test#TestReconcileOrigin_Ended` -> `func TestReconcileOrigin_Ended(t *testing.T)`

**Edit:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/fabricengine#EndedTeardown`

**Uses:**
- `plan:internal/fabricengine#SnapshotTagged`
- `internal/fabricengine#ClearPushFailure`
- `internal/fabricengine#ListFailedPushes`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D6's ended entries inside the reconcile, and D10's reconcile-side record clearing.
An entry is ended on this machine when it is in the ended set and this machine holds a batten run record of that slug, judged from the local records before any move; a run whose local state shows its Finalize past its parent-side merge (`finalizeEntered`, card 4) is landing here and is not ended; a pair with no batten run record is never classified, and an entry only local Bolt holds is never ended.
`reconcileEnded` reads no `finalizeEntered` itself: it receives the filtered `EndedSet` card 16 computes right after `readEndedSet`, before Bolt's replay and any move, so a parent the same reconcile fast-forwards to another machine's landing never makes that landing read as this machine's.
`reconcileEnded` runs in D3 step 4, applying D6 to an entry before its branches are judged: for each ended entry with a local part of its pair (the task worktree path, the sibling worktree path, either local branch, or its launchers, the remnants `refuseWhenNothingRemains` counts as local) it calls `OriginSeams.Teardown` with an `EndedTeardown` naming the slug and, as `EndedTeardown`'s new `EarlierSnapshot` field, whether `SnapshotTagged` finds the snapshot tag an earlier force-case run left (the seam decides the wake-hold half of the force case itself, card 18).
An ended entry with no local part of its pair is reported `ended` with no teardown, no error and no network call: `Topology.RemoveRefusal`'s `refuseWhenNothingRemains` would ask origin for each side and answer `ErrPairNotFound`.
A second machine reaches that state while the landing machine's run record still sits on the prime weft, until the landing machine's cleanup removes it, so such an entry recurs at every reconcile; it holds no action, so the report keeps it and `Summary` and card 23's wake notice omit it.
`reconcileEnded` appends each `TeardownReport`'s mutations to the report's record (`rec.Extend`), so a removed pair, its tags and its deleted branches appear in the top-level `mutations` array of orch start's and cleanup's envelopes (cards 19 and 20) and in the wake's Info log lines (card 23); the run-scope callers (cards 21 and 22) tear nothing down and log only their moves.
The removal is pairteardown's local mode (card 14): pending records committed, a local `set-aside/<slug>/<sha>` tag instead of a pushed archive tag, no origin step, no board write.
It writes no commit of its own (the board entry's local-removal request is departed from, since a local removal commit would be pushed over origin by the next push); the prime weft reaches origin's state of the record through D5's replay.
The entry is reported `ended` as landed (`done`) or removed from the board (`removed`), with every tag; a teardown reporting busy (a busy driver, a held teardown or weft write lock, or a detached push holding the teardown lock) is `ended` with the busy flag and not removed, its outcome naming the way forward: for a busy driver, the removal waits for that driver to end, which the run-scope reconcile at its next Merge-In or Finalize makes certain by stopping it with Halt (card 22), and `lyx cleanup --apply` re-run once it has ended removes the pair; for a held lock, `lyx cleanup --apply` re-run once its holder has released it; under a wake-written hold the force case (card 18) ends the driver itself, so there the busy reading is a held lock and the way forward is the bare re-run card 23's notice names; a teardown failing with any other error is `ended` with that error on its outcome, not removed and stopping no other entry, so the report names it and a later `lyx cleanup --apply` retries the removal.
The run scope classifies its own run as ended like any other but never calls the teardown on it, since its driver is the caller (PATTERN-batten-bookend).
A judge-only scope reports the teardowns it would run and runs none.
After the branches are judged, the reconcile removes every failed-push record whose branch is no longer local (`ListFailedPushes`, `ClearPushFailure`).
Bound: the only waits are those the teardown seam's local mode makes for its own loop kill, at most two `runLockWait` per ended pair and their sum over a hub's ended pairs.

`TestReconcileOrigin_Ended` (integration, `hubforge`, `boardkit.InstallReplay` called first, a recording teardown fake, run without `t.Parallel` since it sets the call hook; the local removal's own effects are card 13's and card 14's) rows: an entry done on origin with a local pair is handed to the teardown seam once with its slug, and the seam's report maps to `ended` as landed with the report's tags, the mutation the recording fake returns appearing in the `OriginReport`'s record; an ended entry with a run record and no local pair is `ended` with no seam call and no error, the call hook still counting two ls-remote calls; in hub scope, a pair status with `current_producer` `Finalize` and no history row, its entry done on origin and the local parent behind origin, is `ended` and handed to the teardown seam although the same reconcile fast-forwards the parent; an entry removed from origin's board is `ended` as removed; the same entry with a teardown reporting busy is `ended` with the busy flag and not removed, its outcome and `Summary` naming the wait for the driver to end and `lyx cleanup --apply`; a teardown returning another error is `ended` with the error, and a second ended entry in the same reconcile is still handed to the seam; an entry only local Bolt holds, with a parked pair, is not ended and not torn down; a run whose pair status holds a `Finalize` history row is not ended; under run scope, a pair status whose `current_producer` is `Finalize` with no history row, its local parent branch not holding the task's work and its entry done on origin, is `ended` as landed, the outcome card 22 stops with Halt, while the same status over a parent the run's own Finalize merged into is not ended, the re-entry after its done mark; a pair with no batten run record and no board entry is never removed; run scope over its own ended run reports it ended and calls no teardown; judge-only reports the would-be teardown and calls none; a failed-push record of a branch no longer local is removed; an earlier snapshot tag sets `EarlierSnapshot`.

**ImpactSummary:** Extends the one new entry point and one of its types; no caller exists until card 19.

**Commit:** 17: fabricengine: the origin reconcile tears down runs that ended elsewhere
