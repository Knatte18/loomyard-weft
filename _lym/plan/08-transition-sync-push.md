# Card 8 — transition-sync-push

**Create:**
- `plan:internal/fabricengine#IsPushOwed` -> `func IsPushOwed(err error) bool`
- `plan:internal/fabricengine_test#TestIsPushOwed` -> `func TestIsPushOwed(t *testing.T)`
- `plan:internal/statuscommit#owedRetryCeiling` -> `const owedRetryCeiling = time.Minute`
- `plan:internal/statuscommit#TestNew_OwedPushRetriesBehindTheGate` -> `func TestNew_OwedPushRetriesBehindTheGate(t *testing.T)`

**Edit:**
- `internal/statuscommit#Deps`
- `internal/statuscommit#New`
- `internal/statuscommit#TestNew_OrdinaryPath`
- `internal/statuscommit#TestNew_CommitErrorPropagates`
- `internal/statuscommit#TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`
- `internal/statuscommit#TestNew_PushErrorReturnsNil`
- `internal/statuscommit#TestNew_MergeActiveSkips`
- `internal/battencli#commitStatusDeps`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#newCommitStatusSeam`
- `internal/battencli#commitRunRecords`
- `internal/battencli#commitCancelRecord`
- `internal/battencli#TestNewCommitStatusSeam`
- `internal/battencli#TestCommitRunRecords`
- `internal/battencli#TestCommitCancelRecord`
- `internal/battencli#TestStatusPushSpawns_TellTheirLocationAndGateSlug`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#newCommitStatusSeam`
- `internal/loomcli#TestNewCommitStatusSeam_OrdinaryPath`
- `internal/loomcli#TestNewCommitStatusSeam_BoardStatus`
- `internal/loomcli#TestNewCommitStatusSeam_StoppedCommitStartsNoPush`
- `internal/loomcli#TestCommitStatusSeam_Real`
- `internal/fabricengine/doc.go`
- `cmd/lym#unboundedWaitFiles`

**Uses:**
- `plan:internal/pairteardown#PushRun`
- `internal/pauserecord#Admit`
- `internal/loomcli/doc.go`

**Intent:**
A shed transition pushes its records before the run acts (D1).
`statuscommit.Deps` drops `Spawn` and gains:
`Push func() error` (the synchronous push of the run's records, and the code side in a task pair; required, `New` panics on a nil one as a wiring guard),
`Admit func() (release func())` (the network gate admission; nil takes none),
`Owed func(err error) bool` (whether a failed push is owed and retried; nil reads nothing owed)
and `Sleep func(time.Duration)` (the retry wait, injected so no test waits it out; nil is `time.Sleep`).

`New`'s closure, after the mid-merge skip, takes `Admit` before the commit, so a pause or hold written after the commit stops the next producer and never the push's first attempt, which runs under the admission.
It commits as today; a commit that made no commit releases the admission and returns.
It then pushes:
- landed: release, return nil, and the next producer starts;
- `Owed` true (an unreachable origin, a held push lock or a teardown wait past `StatusPushLockWait`): release the admission, so the retries wait at the gate like any network call and a pause holds them,
  then retry from one second, doubling to `owedRetryCeiling` while the push stays owed, until it lands; the failed-push record and its notice stand meanwhile and `ReadRun` reads it (card 7);
- any other error, a hand-over (`ErrHandedOver`) included: release and return it, so the run halts at the transition with the failure recorded.
The wait is unbounded and lives as long as the run, so `PATTERN-daemon-wakeups` binds it: one-second floor, backoff to the ceiling;
`internal/statuscommit/statuscommit.go` joins `unboundedWaitFiles`, so the guard scans it for sub-second literals, and card 14 lists it in the pattern's background.
`fabricengine.IsPushOwed` reports the owed classes: a `*PushFailedError` of class `transport` or `teardown_lock`, or `ErrPushLockBusy`.
A bare `ErrPairPushRunning` is not owed: it comes only from a push with a zero `TeardownWait`, which writes no record, so `ReadRun` could not see it; every transition seam passes a positive wait where it takes the lock.
The package comment and `New`'s doc state the new dispositions in place of spawn-warns.

The seams:
- batten (`battenCommitStatusDeps`) pushes the prime's records side with `pairteardown.PushRun(location, location, runID, RunPush{LockWait: StatusPushLockWait})`, outside the pair, so a hand-over removes the local pair.
  The prime's records push takes no teardown lock (`statusPushRequest` sets no teardown slug for the prime), so it passes no `TeardownWait`.
  `commitRunRecords` pushes through `Push` too.
  `commitCancelRecord` pushes through `Push`, and an owed push (`IsPushOwed`) is logged and returns nil with the commit made, so `lym batten cancel` goes on to end the run's processes and owes its push (card 9); any other push error is returned.
  A nil `Push` in battencli's `commitStatusDeps` pushes nothing, for a receiver a test builds by hand.
  battencli's package comment, the header of `commitstatus.go`, says the push is run and waited for, in place of "the push is spawned, not run".
- loom (`commitStatusDepsGuardedBy`) wires the transition's push to the same `Push` it gave commit-records in card 6, adding `TeardownWait: StatusPushLockWait`; its `Spawn` field goes.
Both wire `Admit` to `pauserecord.Admit("transition push")`, `Owed` to `fabricengine.IsPushOwed` and `Sleep` to `time.Sleep`.
So the pull-request record and the fixer attempt counts, which the run's status pathspec carries (`LoomDurableDirRel`), are on origin before the next producer starts (D1's tracked writes).
The battencli `spawnStatusPush` var stays for batten cleanup; loomcli's stays for the pull-request record until card 9.

Tests:
- `TestNew_OwedPushRetriesBehindTheGate` (tier0): a fake `Admit` counts outstanding admissions; a push owed twice then landing:
  the first push runs with one admission outstanding, every retry runs with none, the sleeps are 1s then 2s, and the closure returns nil after the landing;
  a ceiling row asserts the wait stops doubling at a minute.
  The pause half of D1's criteria rests on this: the first attempt cannot be held, and every retry passes the gate unadmitted, where `pauserecord.Network` holds a covered process (`TestNetwork`).
- `TestIsPushOwed` (tier0): rows for a transport and a `teardown_lock` `*PushFailedError` and `ErrPushLockBusy` (owed), a `not_fast_forward` failure, a bare `ErrPairPushRunning` and `ErrHandedOver` (not owed).
- `TestNew_PushErrorReturnsNil` becomes the not-owed row: the error is returned; the other `TestNew_*` tests replace `Spawn` with `Push` and assert `Admit` is taken once per committed transition and released.
- `TestNewCommitStatusSeam`, `TestCommitRunRecords` (battencli, tier0) count pushes in place of spawns; `TestCommitCancelRecord` does too and gains an owed-push row returning nil with the commit made;
  `TestStatusPushSpawns_TellTheirLocationAndGateSlug` drops its transition-seam row.
- loomcli's seam tests (tier0) do the same.
  `TestCommitStatusSeam_Real` (tier1) drops its `spawnStatusPush` swap and the comment saying the seam spawns its push, since the seam now pushes;
  it gains a row: after the seam returns, origin's records branch holds the transition and a fixer attempts file written under the durable directory, and origin's code branch the code commit.

**ImpactSummary:** Turns both shed status seams from a detached spawn into a synchronous push with an owed retry; `statuscommit`'s two callers and their tests change, the guard's wait list gains one file, and no other module calls the core.
