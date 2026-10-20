# Card 17 — origin-reconcile-ended

**Create:**
- `plan:internal/fabricengine#reconcileEnded` -> `func reconcileEnded(l *lyxcwd.Location, scope ReconcileScope, seams OriginSeams, ended EndedSet, res *OriginReport) error`
- `plan:internal/fabricengine_test#TestReconcileOrigin_Ended` -> `func TestReconcileOrigin_Ended(t *testing.T)`

**Edit:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/fabricengine#EndedTeardown`

**Uses:**
- `plan:internal/fabricengine#SnapshotTagged`
- `plan:internal/fabricengine#ClearPushFailure`
- `plan:internal/fabricengine#ListFailedPushes`
- `plan:internal/fabricengine#finalizeEntered`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D6's ended entries inside the reconcile, and D10's reconcile-side record clearing.
An entry is ended on this machine when it is in the ended set and this machine holds a batten run record of that slug, judged from the local records before any move; a run whose local status history shows Finalize entered (`finalizeEntered`) is landing here and is not ended; a pair with no batten run record is never classified, and an entry only local Bolt holds is never ended.
`reconcileEnded` runs in D3 step 4, applying D6 to an entry before its branches are judged: for each ended entry it calls `OriginSeams.Teardown` with an `EndedTeardown` naming the slug and, as `EndedTeardown`'s new `EarlierSnapshot` field, whether `SnapshotTagged` finds the snapshot tag an earlier force-case run left (the seam decides the wake-hold half of the force case itself, card 18).
The removal is pairteardown's local mode (card 14): pending records committed, a local `set-aside/<slug>/<sha>` tag instead of a pushed archive tag, no origin step, no board write.
It writes no commit of its own (the board entry's local-removal request is departed from, since a local removal commit would be pushed over origin by the next push); the prime weft reaches origin's state of the record through D5's replay.
The entry is reported `ended` as landed (`done`) or removed from the board (`removed`), with every tag; a teardown reporting busy (a busy driver, a held teardown or weft write lock, or a detached push holding the teardown lock) is `ended` with the busy flag and not removed; a teardown failing with any other error is `ended` with that error on its outcome, not removed and stopping no other entry, so the report names it and a later `lyx cleanup --apply` retries the removal.
The run scope classifies its own run as ended like any other but never calls the teardown on it, since its driver is the caller (PATTERN-batten-bookend).
A judge-only scope reports the teardowns it would run and runs none.
After the branches are judged, the reconcile removes every failed-push record whose branch is no longer local (`ListFailedPushes`, `ClearPushFailure`).
Bound: the only waits are those the teardown seam's local mode makes for its own loop kill, at most two `runLockWait` per ended pair and their sum over a hub's ended pairs.

`TestReconcileOrigin_Ended` (integration, `hubforge`, `boardkit.InstallReplay` called first, a recording teardown fake; the local removal's own effects are card 13's and card 14's) rows: an entry done on origin with a local pair is handed to the teardown seam once with its slug, and the seam's report maps to `ended` as landed with the report's tags; an entry removed from origin's board is `ended` as removed; the same entry with a teardown reporting busy is `ended` with the busy flag and not removed; a teardown returning another error is `ended` with the error, and a second ended entry in the same reconcile is still handed to the seam; an entry only local Bolt holds, with a parked pair, is not ended and not torn down; a run whose pair status shows Finalize entered is not ended; a pair with no batten run record and no board entry is never removed; run scope over its own ended run reports it ended and calls no teardown; judge-only reports the would-be teardown and calls none; a failed-push record of a branch no longer local is removed; an earlier snapshot tag sets `EarlierSnapshot`.

**ImpactSummary:** Extends the one new entry point and one of its types; no caller exists until card 19.

**Commit:** 17: fabricengine: the origin reconcile tears down runs that ended elsewhere
