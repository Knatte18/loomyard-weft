# Card 12 — board-test-moves

**Create:**
- `plan:internal/boardengine_test#TestReconcileOrigin_EndedRuns` -> `func TestReconcileOrigin_EndedRuns(t *testing.T)`
- `plan:internal/boardengine_test#TestReconcileOrigin_OwnedElsewhere` -> `func TestReconcileOrigin_OwnedElsewhere(t *testing.T)`
- `plan:internal/boardengine_test#TestPush_BoardReplay` -> `func TestPush_BoardReplay(t *testing.T)`
- `plan:internal/boardengine_test#writeRunStatus` -> `func writeRunStatus(tb testing.TB, prime *lymcwd.Location, slug, producer string, history ...string)`
- `plan:internal/boardengine_test#teardownRecorder` -> `type teardownRecorder struct`
- `plan:internal/boardengine_test#requireTrackingRefPrunedWithUpstreamSet` -> `func requireTrackingRefPrunedWithUpstreamSet(t *testing.T, worktree, branch string)`
- `plan:internal/gitkit#CommitOnBare` -> `func CommitOnBare(tb testing.TB, bare, branch, rel, content, message string) string`
- `plan:internal/testkit/boardkit#BoardFile` -> `func BoardFile(entries ...string) string`
- `plan:internal/fabricengine#RunStatusFileForTest` -> `func RunStatusFileForTest(prime *lymcwd.Location, slug string) string`

**Delete:**
- `internal/fabricengine_test#TestReconcileOrigin_Ended`
- `internal/fabricengine_test#teardownRecorder`
- `internal/fabricengine_test#endedTemplate`
- `internal/fabricengine_test#arrangeEnded`
- `internal/fabricengine_test#endedHub`
- `internal/fabricengine_test#writePairStatus`
- `internal/fabricengine_test#publishBoard`
- `internal/fabricengine_test#originBoard`
- `internal/fabricengine_test#requireTrackingRefPrunedWithUpstreamSet`

**Edit:**
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/fabricengine_test#movesPair`
- `internal/fabricengine_test#writeStatus`
- `internal/fabricengine_test#newFailureFixture`
- `internal/fabricengine_test#TestBoltReplayers`
- `internal/fabricengine_test#plantRunTransition`
- `internal/fabricengine_test#recordsFixture.originCommit`
- `internal/fabricengine_test#originCommitFile`
- `internal/testkit/boardkit#WriteBoard`

**Extends:**
- `internal/boardengine/doc.go`
- `internal/gitkit/doc.go`

**Uses:**
- `internal/testkit/boardkit#InstallReplay`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#CoalescePushRequest`
- `internal/fabricengine#ListFailedPushes`
- `internal/gitkit#RevParse`
- `internal/gitkit#BranchExists`
- `internal/gitkit#CommitFile`
- `internal/gitkit#Git`
- `internal/lymcwd#ResolveWorktree`
- `internal/pairteardown#New`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#OwnerReader`
- `internal/pairteardown#Teardown.OriginSeams`
- `internal/boardengine#ParseBoardTasks`
- `internal/fabricengine/doc.go`

**Intent:**
D12's moves of this concern's tests: what rests on the board's replay moves to `internal/boardengine`, and this concern's fabric tests stop importing `internal/testkit/boardkit`, `internal/boardengine`, `internal/shedrun` and `internal/shedengine`,
but for the shared transition-planting helper file below.

A `ReconcileOrigin` or records push reads the board's run ledger only when a recorded run, an owner question or a Bolt replay needs it (`readOriginEnded`, `readOriginOwners`, `retryBoltPush`).
So a fabric test that records no run, asks no owner and diverges no Bolt branch needs no replayer, and the rows that need one pin the board module's ended set, owners or entry merge.

Moved, as tier1 tests in package `boardengine_test`, which imports `internal/hubforge`, `internal/pairteardown` and `internal/testkit/boardkit` with no cycle, being an external test package, and which register the real replay through `boardkit.InstallReplay`:
- `TestReconcileOrigin_EndedRuns` takes `TestReconcileOrigin_Ended`'s rows that pin which runs origin's board ends:
  done entry with a local pair torn down once and reported landed; ended with no local pair; Finalize before the parent merge; past Finalize; the four Raddle rows; entry removed; local-only Bolt entry; pair with no run record; the run-scope rows; and the vanished branch's failed-push record cleared only by a moving reconcile.
  It writes run records through `writeRunStatus`, shed's own types written at shed's path.
  It keeps the earlier-snapshot-tag row, which pins the engine's `SnapshotTagged` read.
  It tears pairs down through the real seam `teardownRecorder` wraps, and asserts the reconcile's exact printed `mutations` array, holding the entries the real teardown records (D4, D5);
  it asserts no tags, kept lines or mutation entries of a made-up report, since card 10's `TestEndedTeardowns` holds the copying of a report.
  The busy, failing-teardown and judge-only rows are deleted, held by card 10's `TestEndedTeardowns`.
- `TestReconcileOrigin_OwnedElsewhere` takes `TestReconcileOrigin_Moves`' owned_elsewhere rows: the summary and way forward per subject and scope, the records branch left unreplayed while fast-forwards move, and ended-and-owned-elsewhere; and `TestReconcileOrigin_Judge`'s entry-with-a-run-record and `PairMayBeAbsent` entry rows, and the Bolt rows (needs_replay of a diverged Bolt, the Bolt replay that sets aside an ended entry's status, the dirty Bolt that stops every entry, the both-sides prime weft records change).
  Its rows take `OwnerOf` from `pairteardown.OwnerReader` with a fixed identity and a `parentOf` naming each row's parent, over the owners the board's own entries record, so no row scripts an owner answer:
  the beam and root subject rows get their subjects from the real reader's adopt-subject read over that `parentOf`.
  The no-origin-head row observes that the reader was never asked through a wrapper that records each call and passes it to the real reader.
  It keeps the nil-`OwnerOf` and `OwnerOf`-error rows, which pin `readOriginOwners`' early return and its wrapped seam error over a recorded run;
  the error row provokes the real reader's error with an owner read over board bytes that do not decode.
  The empty-subject row, whose way forward names the slug, does not move: card 9's `TestClassifyEntries` holds that rule as a row.
- `TestPush_BoardReplay` takes `TestPushFailureClasses`' `ABoltPushRejectedOverAnUnrelatedOriginEntryIsReplayedAndLands`, `ABoltPushRejectedWhileTheBoardLockIsHeldDefersWithADeferredRecord`, `APrimeWeftPushRejectedAfterThisMachinesFinalizeKeepsTheLandingsCommits`, `ADetachedWeftPushRejectedAsNotFastForwardIsReplayedAndPushed` and `AParkedPushOfABranchGoneOnOriginDoesNotRecreateIt`;
  `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`'s `ALandingCommitPastOriginHoldsTheRecordsUntilTheRunIsDone`;
  and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`' two `OwedUnrecorded_*OfAnEndedEntry*` rows.
  Each keeps its tier and drops every bare side spelling the vocabulary scan refuses outside fabric's owner set, using `hubforge` accessors in place of side-named helpers.

Rewritten in place:
- `TestReconcileOrigin_Moves` and `TestReconcileOrigin_Judge` drop `boardkit.InstallReplay`, `boardengine.RunStatus`, `shedrun.StatusFile` and `shedrun.PrimeRunLock`;
  `movesPair`'s template writes no run status, so the kept rows record no run and read no ledger.
  Each kept row of cards 9 and 11 that asserted its entry (the clean and dirty pair weft, the live pair, the held weft lock, the held Bolt push lock) asserts its branch outcome instead, state and way forward,
  since the entry's classification over those states is `TestClassifyEntries`' rows.
- `TestBoltReplayers`' row "With no replayer registered the ended-set read and the owner read fail with ErrReplayerMissing" writes its own run record through `writeStatus` before its reconcile, so it still reaches both ledger reads.
- `writeStatus` takes the status path from the installed run-records seam through `RunStatusFileForTest` (`export_test.go`) in place of `shedrun.StatusFile`; `newFailureFixture` drops `boardkit.InstallReplay`.

Support for the moved tests:
- `PATTERN-test-isolation` puts support two packages share in one kit and bars a test package's own `hubforge` fixture type, and `PATTERN-test-stand-ins` bars adding another notifier fake.
  So the moved rows take a `*hubforge.Hub` and read its accessors, and no fixture type or notifier fake is created in `boardengine_test`.
  A helper both packages need that uses neither git nor `hubforge` lives in one kit.
  Shared support is never restated per package, and a kit may not import `internal/gitkit` (`internal/testkit/enforcement_test.go`'s `kitDeniedImports`),
  so the git plumbing both packages need lives in `internal/gitkit`, the leaf `PATTERN-leaf-packages` names for test git plumbing.
- `gitkit.CommitOnBare` (new) clones `bare` at `branch` into a temp directory, commits `rel` with `content` and `message`, pushes it and returns the new tip.
  Fabric's `recordsFixture.originCommit` and `originCommitFile` call it in place of their own clone, commit and push, so one copy stands.
  `originCommit` sits in `replay_integration_test.go`, another concern's file, so that edit departs from D9's own-files criterion;
  `PATTERN-test-isolation`'s rule against support duplicated per package forces it, and the commit message names the file and that cause.
- Fabric's push and reconcile assertion helpers stay as they are, and no kit holds assertions (`PATTERN-test-isolation`'s Enforcement).
  Each moved row asserts inline over `fabricengine.ListFailedPushes` and the report's `Entries`, and counts backoffs with an inline closure, adding no helper to `boardengine_test`.
- The rows that read git read through `internal/gitkit`: a bare branch's tip through `gitkit.RevParse` of `refs/heads/<branch>`, a bare branch's presence through `gitkit.BranchExists`,
  a landing commit through `gitkit.CommitFile` with the landing trailer in its message, and tags through `gitkit.Git`'s `tag --list`.
  `ALandingCommitPastOriginHoldsTheRecordsUntilTheRunIsDone`'s reads through `fabricengine.BareBranchSHAForTest`, an accessor only fabric's own test binary compiles, become `gitkit.RevParse` reads too.
  A row reads a pair's branch names through `gitkit.CurrentBranch` of the hub's `PairCodeWorktree` and `PairRecordsSibling`.
  Fabric's `tagsOf`, `landOn`, `originHasBranch` and `boardFileOf` sit in other concerns' files (D9) and stay there.
- The moved rows write board files as the entry JSON objects `boardkit.WriteBoard` takes, encoded by `boardkit.BoardFile`, which `WriteBoard` now writes, and read entry titles through `boardengine.ParseBoardTasks`.
- The moved rows register the board's replay through `boardkit.InstallReplay` and build their hub through `hubforge.CopyHub`.
  They add a pair through `hubforge.AddPair`, which pushes both sides, and resolve its location through `lymcwd.ResolveWorktree`.
  They publish the prime's records with one `gitkit.Git` `push --set-upstream`,
  and write origin's board, an entry marked done included, through `gitkit.CommitOnBare` over `boardkit.BoardFile`.
  No moved row changes `heddle.json`, so none registers heddle's replayer; a row that changed it would register it in its own body.
- `writeRunStatus` writes a run record of `producer` and the `history` rows through `state.UpdateJSON` over shed's `Status` and `HistoryEntry` types, at `shedrun.StatusFile` under `shedrun.StatusLock`.
  `APrimeWeftPushRejectedAfterThisMachinesFinalizeKeepsTheLandingsCommits` writes its pair record and its prime record that way, at shed's path for each location, and commits the prime record it writes;
  no moved row marshals a shed record by hand (D12).
- `requireTrackingRefPrunedWithUpstreamSet` moves: its only callers are the two moved `OwedUnrecorded` rows.
- Moved rows pass `SyncOptions` with no `Notifier` and drop their mail-line assertions.
  No notifier is installed in the `boardengine_test` binary, so a failed push writes its record and mails nothing (`recordPushFailure`'s nil-notifier return); card 5's `TestMergeFailedPush` notify rows hold the mail rule.
- `teardownRecorder` serves the moved reconcile rows as a recording wrapper with no reply field: it records each `EndedTeardown` and passes it to the real seam.
  The rows build that seam with `pairteardown.New(h.Location)` and its `OriginSeams(pairteardown.HubSeamOptions(h.Location))`, and set `LivePair` to nil on the returned seams, which `OriginSeams` always fills.
  Fabric's `teardownRecorder` goes with `TestReconcileOrigin_Ended`, its last user in fabric.
  Unverified: that the real teardown of these pairs runs in tier1 with no tmux server, and which board fields the real owner reader needs.
  The fabric helpers whose last users move go: `endedTemplate`, `arrangeEnded`, `endedHub`, `writePairStatus`, `publishBoard` and `originBoard`.
- `TestPushFailureClasses`, `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` drop `boardkit`; a kept row that still reads the ledger is a defect of this card's split and moves with the rows above.
- `plantRunTransition` moves from `codetip_integration_test.go` to a new helper file of its own, `internal/fabricengine/transitionplant_helper_test.go`, the planting support this concern's tests and the code-tip settle's share,
  so no other concern's file changes for it and `codetip_integration_test.go` drops its `shedrun` and `shedengine` imports; its file header drops the sentence on holding the helper.
  The helper keeps planting its status record, and its file keeps the shed imports that needs:
  `TestReturnToPreviousTransition` decodes the committed status and asserts the planted machine (`codetipsettle_integration_test.go`), so a plant with no record would break it.
  The pair-push owner-guard rows keep calling it in the same package; they read the transition's code-tip trailer and the reflog (`resetUnnamedTip`, `unmadeOriginCommit`), never the planted machine, so none moves.

The commit message lists every deleted test and row and, for each moved or kept test, the rule it alone guards (overview, Shared Decisions).
The boardengine doc addition says the package's tests hold fabric's flows that rest on the board's replay.
The gitkit doc addition names `CommitOnBare` among the commit helpers: a commit pushed to a bare repository from a second clone, as another machine makes it.

**ImpactSummary:** Test-only: three tests join internal/boardengine, gitkit gains a commit-on-bare helper, boardkit gains its board-file encoder, and this concern's fabric test files lose their board and shed imports; no package outside the test support packages and test files changes.
