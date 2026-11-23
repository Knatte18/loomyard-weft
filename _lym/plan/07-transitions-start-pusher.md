# Card 7 — transitions-start-pusher

**Edit:**
- `internal/statuscommit#Deps`
- `internal/statuscommit#New`
- `internal/statuscommit#TestNew_OrdinaryPath`
- `internal/statuscommit#TestNew_CommitErrorPropagates`
- `internal/statuscommit#TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip`
- `internal/statuscommit#TestNew_MergeActiveSkips`
- `internal/statuscommit#TestNew_NilPushPanics`
- `internal/battencli#commitStatusDeps`
- `internal/battencli#commitRunRecords`
- `internal/battencli#battenCommitStatusDeps`
- `internal/battencli#commitStatusMarker`
- `internal/battencli#newCommitStatusSeam`
- `internal/battencli#battenCLI.seedFreshStatus`
- `internal/battencli#TestNewCommitStatusSeam`
- `internal/battencli#TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`
- `internal/battencli#TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`
- `internal/battencli#TestCommitRunRecords`
- `internal/battencli#TestFreshStartCommitsWorktreeCreateThroughTheSeam`
- `internal/battencli#TestBattenRecordsCommitSeams`
- `internal/battencli#newFakeReceiver`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#loomCommitStatusDeps`
- `internal/loomcli#newCommitStatusSeam`
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#loomCLI.commitRecordsCmd`
- `internal/loomcli/doc.go`
- `internal/loomcli#stubRecordsDeps`
- `internal/loomcli#TestCommitRecordsVerb_Success`
- `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes`
- `internal/loomcli#TestNewCommitStatusSeam_OrdinaryPath`
- `internal/loomcli#TestNewCommitStatusSeam_BoardStatus`
- `internal/loomcli#TestNewCommitStatusSeam_Finish`
- `internal/loomcli#TestLoomPark`
- `internal/loomcli#TestPrePushCheckWiredIntoDeps`
- `internal/loomcli#TestCommitStatusSeam_Real`
- `internal/loomcli#TestCommitStatusDeps_SkipsTheBoardStatusAfterAFilingMark`
- `internal/loomcli#TestRecipeKeyedWiring`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `cmd/lym#unboundedWaitFiles`

**Delete:**
- `internal/statuscommit#retryOwedPush`
- `internal/statuscommit#owedRetryInitial`
- `internal/statuscommit#owedRetryCeiling`
- `internal/statuscommit#TestNew_PushErrorReturnsNil`
- `internal/statuscommit#TestNew_OwedPushRetriesBehindTheGate`
- `internal/statuscommit#fakeGate`
- `internal/statuscommit#fakeGate.admit`
- `internal/battencli#TestNewCommitStatusSeam_RetriesAFailedStartChainPush`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#pushFailureWayForward`
- `internal/loomcli#TestCommitRecordsVerb_StoppedCommitPushesNothing`
- `internal/loomcli#TestNewCommitStatusSeam_StoppedCommitStartsNoPush`

**Uses:**
- `plan:internal/runpusher#Start`
- `plan:internal/runpusher#TargetOf`
- `internal/battencli#battenCommitStatusSeam`
- `internal/battencli#commitCancelRecord`
- `internal/battencli#pushRecordsOrOwe`
- `internal/loomcli#loomPark`

**Intent:**
Make every transition and every driver-stop records commit of a batten or loom run commit locally and start its worktree's pusher,
never push (D1, D8).

`statuscommit`:
- `Deps` loses `Push`, `Admit`, `Owed` and `Sleep` and gains `StartPush func()`, which starts the worktree's pusher and logs its own failure.
  `New` panics on a nil `StartPush`.
- `New`'s closure keeps the mid-merge skip and the commit's hard error, then calls `StartPush` after a commit that made one.
  It takes no network-gate admission, pushes nothing, fetches nothing and waits for nothing.
  `retryOwedPush` and its two constants go, and so does the package comment's synchronous push.
- `cmd/lym#unboundedWaitFiles` drops `internal/statuscommit/statuscommit.go`, which no longer waits.

Both seams build their target the same way, `runpusher.TargetOf(location, shedrun.ResolveRunID(location, runID))`,
and start it through `runpusher.Start`: batten's location is the prime, whose target carries no slug (card 5),
and loom's is the run's pair, whose target carries the run's slug.
So neither builder chooses between a prime and a pair target, and `TestStart`'s target rows pin what each gets.

batten:
- `commitStatusDeps` loses `Admit`, `Owed` and `Sleep` and gains `StartPush func()`.
  It keeps `Push`, its `push` method and `pushRecordsOrOwe`, which only the out-of-scope pushes call:
  the cancel's tracked-intent record (`commitCancelRecord`) and the re-adopt's failed-stop record (`internal/battencli/readopt.go`).
  `battenCommitStatusDeps` keeps filling `Push` for them and fills `StartPush`, logging a failed start at Warn:
  the next commit, tick or resume starts one.
- `newCommitStatusSeam` has one core for every row: start rows, skipped rows and running rows alike commit and start the pusher,
  and the seam never calls `Push`.
  The start core, the `push_pending` mark and the start-row exception of `926b5c28e` go.
- `commitStatusMarker` drops `PushPending`; a marker an older binary wrote with `push_pending` still reads,
  since `state.ReadJSON` refuses no unknown key (`PATTERN-tolerant-records`).
- `commitRunRecords` commits and starts the pusher; it returns only a merge-state or commit failure.
- `battenCLI.seedFreshStatus`'s comment drops the push: the seeded Worktree-Create commit goes through the seam like any transition.
- The file comment of `internal/battencli/commitstatus.go` states the new seam.

loom:
- `commitStatusDeps` loses `Push`, `Admit`, `Owed` and `Sleep` and gains `StartPush func()`.
- `loomCommitStatusDeps` commits unguarded and fills `StartPush`; `commitStatusDepsGuardedBy` goes,
  since neither a transition nor `commit-records` runs the pre-push check: the pusher runs it before each push (D7, card 6).
- `newCommitStatusSeam` drops its stopped-commit branch.
- `commitRecordsVerb` commits and starts the pusher; its envelope is ok with `committed` and `pusher_started`.
  A merge-state probe failure and a commit failure stay error envelopes with their ways forward;
  every push branch goes, with `pushFailureWayForward`.
  The commit always lands when the tree has something to commit.
- `commitRecordsCmd`'s help says it commits and starts the pair's pusher.
- `loomcli/doc.go`'s `commit-records` lines say the same.
- `prePushCheck`, `guardedCommit` and `guardedPush` stay for the pushes out of scope:
  the landing's, and the pull-request record commits `pullRequestRecordCommitter` guards.
- The opening comment of `internal/loomcli/wiring_commitstatus_test.go` names the core's skip and commit dispositions and the pusher start.

A machine adopting the run reads origin's last transition and re-runs that row (D2); adoption itself is unchanged.

Tests, all tier0 unless named:
- `TestNew_OrdinaryPath` asserts a committed transition calls `StartPush` once and an uncommitted one never;
  `TestNew_CommitErrorPropagates`, `TestNew_CommitFailureExplainedByTheReProbeTakesTheSkip` and `TestNew_MergeActiveSkips`
  assert no `StartPush` on their paths; `TestNew_NilPushPanics` panics on a nil `StartPush`.
- `TestNewCommitStatusSeam` rows: a running row, a skipped row, a start row from Seed-Child to Run-Shed and a resumed start row
  each commit with its trailers and start the pusher once, calling no `Push`; a repeated pair commits nothing;
  a marker holding `"push_pending": true` reads and skips the repeated pair.
  The two board-status tests, `TestBattenRecordsCommitSeams` and `newFakeReceiver` drop their `Admit`, `Owed` and `Sleep` fields
  and fill `StartPush`.
- `TestCommitRunRecords` rows: a commit starts the pusher and calls no `Push`; a mid-merge pair or a clean tree starts none;
  a commit failure is returned.
- `TestFreshStartCommitsWorktreeCreateThroughTheSeam` asserts the seeded Worktree-Create commit starts the pusher and pushes nothing.
- `TestCommitRecordsVerb_Success` asserts the ok envelope with `pusher_started`, and `TestCommitRecordsVerb_ErrorEnvelopes` keeps only the probe and commit rows.
- The loom seam tests, `TestLoomPark` and `stubRecordsDeps` drop `Push` and assert `StartPush` where they asserted a push.
- `TestPrePushCheckWiredIntoDeps` keeps only its pull-request row.
- `TestCommitStatusSeam_Real`, `TestCommitStatusDeps_SkipsTheBoardStatusAfterAFilingMark` and `TestRecipeKeyedWiring` are tier1;
  the first asserts the commit lands locally with its trailers and origin is not touched by the seam.
- `TestAdoptIntegration_SecondCloneTakesOver` is tier1 and gains a row:
  the first clone commits one more transition of the run through the seam, which under `go test` starts no pusher, so origin lacks it;
  the second clone adopts and reads the run at the last transition origin holds, and its resume re-runs that row from the top.

**ImpactSummary:** Every batten and loom transition and driver-stop commit stops pushing; the cancel's and re-adopt's record pushes, the landing's, PR-Review's and adopt's are untouched.
