# Card 13 — remove-local-mode

**Create:**
- `plan:internal/fabricengine#ErrPairBusy` -> `var ErrPairBusy error`
- `plan:internal/fabricengine#Topology.removePair` -> `func (t *Topology) removePair(l *lyxcwd.Location, slug string, force, remote, localMode bool) (res RemoveResult, err error)`
- `plan:internal/fabricengine#Topology.RemoveLocal` -> `func (t *Topology) RemoveLocal(l *lyxcwd.Location, slug string, force bool) (res RemoveResult, err error)`
- `plan:internal/fabricengine#Topology.SnapshotUncommitted` -> `func (t *Topology) SnapshotUncommitted(rec *Mutations, l *lyxcwd.Location, slug string) (tag string, paths []string, err error)`
- `plan:internal/fabricengine#SnapshotTagged` -> `func SnapshotTagged(l *lyxcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine#housekeepHook` -> `var housekeepHook func(slug string)`
- `plan:internal/fabricengine#SetHousekeepHookForTest` -> `func SetHousekeepHookForTest(hook func(slug string)) (restore func())`
- `plan:internal/fabricengine#removeAfterLockHook` -> `var removeAfterLockHook func() error`
- `plan:internal/fabricengine#SetRemoveAfterLockHookForTest` -> `func SetRemoveAfterLockHookForTest(hook func() error) (restore func())`
- `plan:internal/fabricengine_test#TestRemoveLocal` -> `func TestRemoveLocal(t *testing.T)`
- `plan:internal/fabricengine#commitRecordsPathsHeld` -> `func commitRecordsPathsHeld(rec *Mutations, weftPath, anchorRel string, relPaths []string, msg string, opts SyncOptions) (sha string, committed bool, err error)`

**Edit:**
- `internal/fabricengine#CommitRecordsPaths`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#commitPendingRecords` -> `func commitPendingRecords(rec *Mutations, l *lyxcwd.Location, slug, warpBranch string, try bool) (committed bool, err error)`
- `internal/fabricengine#CommitPendingRecordsForTest`

**Uses:**
- `plan:internal/fabricengine#setAsideTag`
- `plan:internal/fabricengine#KindTagCreated`
- `plan:internal/fabricengine#ClearPushFailure`
- `internal/fabricengine#regenerateHubWorkspace`
- `internal/fabricengine#WeftWriteLockPathForTest`

**Intent:** D6 step 1's local mode at the fabric layer: a removal that runs no origin step and waits on no other writer.
`Remove`'s body moves into `removePair` with a `localMode` switch; `Remove` calls it with `localMode` false and behaves as today, and `RemoveLocal` calls it with `localMode` true and `remote` false.
In local mode, after the same refusals (every `RemoveRefusal` check still applies, force answering dirtiness only):
- the teardown lock is taken by one try-acquire, a held lock returning an error wrapping `ErrPairBusy` with nothing changed, where `Remove` waits up to the topology's teardown wait;
- `commitPendingRecords` runs with `try` true, where `Remove`'s call keeps the blocking acquire: `CommitRecordsPaths`' body after its lock moves into `commitRecordsPathsHeld`, and `CommitRecordsPaths` takes the weft write lock with its blocking acquire and calls it, as today; with `try` true `commitPendingRecords` takes the weft write lock itself by one try-acquire (a held lock wraps `ErrPairBusy`, nothing changed) and calls `commitRecordsPathsHeld` under it, never `CommitRecordsPaths`, whose second acquire of the same lock file would block;
- instead of `archiveWeftTip`'s pushed archive tag it puts a local `set-aside/<slug>/<sha>` tag on the weft tip (`setAsideTag`) and reports it in a new `RemoveResult.SetAsideTag`;
- for a pair with a local part, no origin step runs and no ls-remote either (a pair with nothing local still meets `RemoveRefusal`'s `refuseWhenNothingRemains`, which asks origin, and card 17 never hands such a pair to the local removal): `removePair` in local mode never takes `removeSibling`'s branch that calls `remoteHeadTip` for a pair with neither sibling worktree nor sibling branch, and treats that sibling as already gone locally; and a failure after the teardown lock skips `respawnPairPush`, the detached push `Remove` respawns for a surviving worktree, leaving that worktree's commits local for the next reconcile;
- the hub workspace file is regenerated with one try-acquire of its lock (`regenerateHubWorkspace` with a zero wait), a held lock skipping it with the `lyx fabric reconcile` warning, reported in a new `RemoveResult.WorkspaceSkipped` (beside two more new fields, `SnapshotTag` and `SnapshotPaths`, which card 14 fills);
- `housekeepStores` is skipped; `Remove`'s call of it first runs `housekeepHook`, nil in production, so a test observes whether the housekeeping ran.
A warp branch kept for unlanded work is reported with `Remove`'s kept reason, as today.
Both modes call `ClearPushFailure` for the pair's code and records branches once the pair is removed, D10's teardown clearing.

`SnapshotUncommitted` serves D6's force case: it try-acquires the weft write lock (held: `ErrPairBusy`, nothing written), commits every uncommitted path of the task worktree and every uncommitted path of the weft worktree outside the record pathspec, untracked ones included, into one commit on no branch built through a temporary index, so it moves no branch and touches neither worktree's index, tags it with an annotated `set-aside/<slug>/<sha>` tag whose message marks it a snapshot, records the commit and the tag in `rec` so the removal's result carries them, the tag as a `KindTagCreated` entry appended only after the tag exists, as `setAsideTag` appends it, releases the lock once the tag is written and returns the tag and the paths; with nothing uncommitted it writes nothing and returns an empty tag.
`SnapshotTagged` reports whether such a snapshot tag of the slug exists, which card 14 reads to tell an earlier run of the force case.
Bound: local mode skips only the origin steps, the board write (it never had one at this layer) and the store housekeeping; the commits the pushed archive tag would have carried are kept under the local tag.

`removeAfterLockHook`, nil in production, runs in `removePair` right after the teardown lock is taken, and its error is returned as the step's failure, so a test fails a removal while both worktrees survive; the existing `removeAfterTeardownHook` runs only after the worktrees are gone and returns nothing, so it cannot.
`SetHousekeepHookForTest` and `SetRemoveAfterLockHookForTest` live in `export_test.go`, and the held-weft-write-lock rows reach the lock's path through the existing `WeftWriteLockPathForTest` shim over the pair's `Fabric`; the hook swaps are package-global, so `TestRemoveLocal` runs without `t.Parallel`.
`TestRemoveLocal` (integration, `hubforge`) rows: a pair with local-only weft commits and pending records is removed with the records committed, a `set-aside/` tag on the weft tip, no tag or branch change on either bare; a held teardown lock and a held weft write lock each return `ErrPairBusy` at once with the pair intact; a `RemoveLocal` whose step after the teardown lock fails with both worktrees surviving, through `removeAfterLockHook`, returns the error with the weft bare unchanged; a held hub workspace lock removes the pair with `WorkspaceSkipped` set and the housekeeping hook never called; a failed-push record of the pair's branch is gone after a `RemoveLocal`, and another after a plain `Remove`; a pair with a task worktree and no sibling, its records origin URL set to `http://127.0.0.1:1/x`, is removed with no remote error; `SnapshotUncommitted` over a dirty task worktree and an untracked weft file tags one commit holding both, both indexes and branches unchanged, its tag in the mutation record, and `SnapshotTagged` then reports true; `SnapshotUncommitted` with the weft write lock held returns `ErrPairBusy` and writes no tag; `SnapshotUncommitted` over a clean pair returns an empty tag and writes nothing.
`CommitPendingRecordsForTest` passes `try` false.

**ImpactSummary:** `Remove` keeps its signature and behaviour; `commitPendingRecords` gains a parameter with two callers, both on this card; `CommitRecordsPaths` keeps its signature and behaviour, its body moved under the lock into `commitRecordsPathsHeld`; `RemoveResult` gains four fields its JSON readers may ignore; and a test hook, `removeAfterLockHook`, nil in production, joins the existing teardown hooks.

**Commit:** 13: fabricengine: a local removal mode with no origin step and no waits
