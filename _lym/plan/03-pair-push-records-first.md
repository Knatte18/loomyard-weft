# Card 3 — pair-push-records-first

**Create:**
- `plan:internal/fabricengine#MachineNames` -> `type MachineNames struct`
- `plan:internal/fabricengine#PairPush` -> `type PairPush struct`
- `plan:internal/fabricengine#PushFailureTeardownLock` -> `const PushFailureTeardownLock PushFailureClass = "teardown_lock"`
- `plan:internal/fabricengine#acquireTeardownShared` -> `func acquireTeardownShared(hubPath, slug string, wait time.Duration) (release func(), acquired bool, err error)`

**Edit:**
- `internal/fabricengine#PushPairAnchored` -> `func PushPairAnchored(l *lymcwd.Location, opts SyncOptions, push PairPush) (res PushResult, err error)`
- `internal/fabricengine#FailedPushWayForward`
- `internal/fabricengine#FailedPush`
- `internal/fabricengine#SyncOptions`
- `internal/fabricengine#pushJob.record`
- `internal/fabricengine/doc.go`
- `internal/fabriccli#addWeftVerbs`
- `internal/beamcli#hubLandDeps`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#landingDeps`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/fabricengine_test#TestPushPairAnchored_PrimeDivergedWeftRemote_ReturnsErrPushRejected`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/fabricengine_test#newPairFixtureAt`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestTeardownLock_SerialisesAddDiscardAndPairPush`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `internal/battencli#TestCleanupIntegration_RefusesOutsideThePrimeThenDryRunsThenRemovesAndPushes`
- `internal/battencli#TestMain`
- `internal/cleanupcli#TestCleanupVerb_RefusesOutsideThePrimeThenListsAndThenRemovesEverySeededLeftover`
- `internal/fabriccli_test#TestMain`

**Uses:**
- `internal/fabriccli/doc.go`
- `internal/beamcli/doc.go`
- `internal/loomcli/doc.go`

**Intent:**
`PushPairAnchored` becomes the pair-push entry the decision record exports: its options move into `PairPush`, so a later part adds the registry's names without changing the signature again.
`PairPush` holds `LockWait` (the push-lock wait, as the old parameter), `TeardownWait` (the teardown-lock wait), `RunSlug` (the run whose record a rejected prime records push reads, card 5) and `Here MachineNames`.
`MachineNames` holds `Current string` and `Former []string`, the names that read as this machine; the caller fills them, since fabric reads no board (D2).
Its doc says its zero value turns the takeover read off; only `CoalescePushBothAt`, a fixture's push, passes it zero (card 5).

The order inverts (D1): the records side is pushed first, and the code side only once the records side landed.
The records side counts landed only when its push returned nil and the records worktree then holds nothing unpushed (`gitrepo.Repo.HasUnpushed` false).
`pushClassified` returns nil for a deferred records push (a held weft write lock or a dirty replay, `pushclassified.go:136`) and for an ended branch (`:423`), so a nil error alone is no landing; this check keeps `pushClassified`'s many callers unchanged.
A records side that fails, is rejected, is deferred or did not land sends no code side.
The error is the records side's, joined with one line naming the code worktree and saying its side was withheld until the records side lands and goes with the next pair push.
The prime still pushes records only and reports `CodePushSkipped` as today.

The teardown lock is taken shared through `acquireTeardownShared`.
A zero `TeardownWait` makes one try, as today.
A positive one waits for the lock's release, polling `lock.TryAcquireReadLock` from one second (a bounded wait, under a minute, outside `PATTERN-daemon-wakeups`'s rule), up to `TeardownWait`.
A wait that runs out records a failed push of class `PushFailureTeardownLock` for the records side and branch, with its creel notice, and returns the error wrapping `ErrPairPushRunning`, so the push is owed (D1).
A later push of that side that lands clears the record as any landed push does.

Every failed-push record names the run it serves: `FailedPush` gains `Run string` (json `run,omitempty`), which `pushJob.record` fills from the new `SyncOptions.RunSlug`.
`PushPairAnchored` sets `RunSlug` to `push.RunSlug` for the prime and to the pair's slug for a pair; every other push leaves it empty, so its records name no run.
Card 7 reads an owed push by this field, so an owed push of the prime's records branch holds only the run that wrote it.
`FailedPushWayForward` gains the class's way forward: the push retries once the removal or push holding the teardown lock lets go, or end the stuck child `lym tender status` lists.

The callers pass `PairPush{LockWait: <their wait>}`, so their behaviour is unchanged but for the order:
`lym fabric push` (`addWeftVerbs`, `LockWaitUnbounded`), the beam push (`hubLandDeps`), the loom `Push` seam and the landing's `PushTask` (`StatusPushLockWait`).
The test callers listed pass `PairPush{LockWait: fabricengine.LockWaitUnbounded}` the same way.
The fabric doc's `PushPairAnchored` paragraph says records first, code after, none after a failed records side, and the teardown wait.

`TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` (tier1) gains rows:
- a records remote that refuses: the call fails, origin's code branch keeps its old tip, and the error names the withheld code side;
- a successful pair push whose mutation record lists the records side's push before the code side's;
- a records side deferred by a held weft write lock: the call returns nil, the records side stays unpushed, and origin's code branch keeps its old tip.
`TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` (tier1) gains two rows over the pair's teardown lock held exclusive:
- held past a short `TeardownWait`: the call returns `ErrPairPushRunning`, nothing reaches origin, and a `teardown_lock` record naming the pair's slug as its run stands for the records side;
- released by the test inside `TeardownWait`: the push waits, then lands both sides, and no record stands.
`TestFailedPushRecord` (tier0) gains the class's way-forward row and a row whose record round-trips `Run`.

**ImpactSummary:** Re-signs one exported push entry with four production callers in fabriccli, beamcli and loomcli and nine test callers across five packages; behaviour changes only in the side order and the teardown wait, which only a positive `TeardownWait` reaches.
