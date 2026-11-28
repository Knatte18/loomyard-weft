# Card 7 — push-side-core

**Create:**
- `plan:internal/fabriccore#OutcomeTakeover` -> `const OutcomeTakeover OutcomeClass = "takeover"`
- `plan:internal/fabriccore#OutcomeProtected` -> `const OutcomeProtected OutcomeClass = "protected"`
- `plan:internal/fabriccore#OutcomeProtectedUnrejected` -> `const OutcomeProtectedUnrejected OutcomeClass = "protected_unrejected"`
- `plan:internal/fabriccore#OutcomeReplayConflict` -> `const OutcomeReplayConflict OutcomeClass = "replay_conflict"`
- `plan:internal/fabriccore#OutcomeOriginAhead` -> `const OutcomeOriginAhead OutcomeClass = "origin_ahead"`
- `plan:internal/fabriccore#OutcomeDeferred` -> `const OutcomeDeferred OutcomeClass = "deferred"`
- `plan:internal/fabriccore#OutcomeRetriesExhausted` -> `const OutcomeRetriesExhausted OutcomeClass = "retries_exhausted"`
- `plan:internal/fabriccore#FieldPushSide` -> `const FieldPushSide Field = "push_side"`
- `plan:internal/fabriccore#FieldPushJudgment` -> `const FieldPushJudgment Field = "push_judgment"`
- `plan:internal/fabriccore#SeamRejectionRetry` -> `const SeamRejectionRetry = "rejection_retry"`
- `plan:internal/fabriccore#PushSideFacts` -> `type PushSideFacts struct`
- `plan:internal/fabriccore#PushSide` -> `func PushSide(snapshot Snapshot) Answer`
- `plan:internal/fabriccore#PushVerdict` -> `type PushVerdict struct`
- `plan:internal/fabriccore#JudgePush` -> `func JudgePush(snapshot Snapshot) PushVerdict`
- `plan:internal/fabriccore#UnseededBoardRefusal` -> `func UnseededBoardRefusal(facts PushSideFacts) *Refusal`
- `plan:internal/fabriccore_test#TestPushSide` -> `func TestPushSide(t *testing.T)`
- `plan:internal/fabricengine#pushJob.read` -> `func (j *pushJob) read(phase fabriccore.Phase, fields []fabriccore.Field, snapshot *fabriccore.Snapshot) error`
- `plan:internal/fabricengine#pushJob.runners` -> `func (j *pushJob) runners(retry func() (deferred bool, err error)) map[fabriccore.ActionKind]actionRunner`
- `plan:internal/fabricengine#errRetryDeferred` -> `var errRetryDeferred error`
- `plan:internal/fabricengine_test#TestPushClassified_Snapshot` -> `func TestPushClassified_Snapshot(t *testing.T)`
- `plan:internal/fabricengine#PushClassifiedSnapshotForTest` -> `func PushClassifiedSnapshotForTest(hubPath, side, path string, opts SyncOptions) (fabriccore.Snapshot, error)`

**Edit:**
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine/doc.go`
- `internal/fabriccli_test#TestInstallRequiredSeams`
- `internal/fabriccore#Snapshot`
- `internal/fabricengine#pushClassified`
- `internal/fabricengine#pushJob`
- `internal/fabricengine#pushJob.checkOrigin`
- `internal/fabricengine#pushJob.classifyFailure`
- `internal/fabricengine#pushJob.failRejection`
- `internal/fabricengine#pushJob.refuseUnseededBoard`
- `internal/fabricengine#pushJob.repushOverOriginContained`
- `internal/fabricengine#outcomeClassOf`
- `internal/fabricengine#TestOutcomeClassOf`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestCoalescePushBothAt_AdvancesBothSidesAndLeavesNoWarpRootLock`
- `internal/fabricengine#TestCoalescePushRequest_CoveredCallDefers`
- `internal/fabricengine#TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`
- `pattern/PATTERN-fabric-boundary.md`

**Delete:**
- `internal/fabricengine#pushJob.retryTransportPush`
- `internal/fabricengine#pushJob.settleRejection`
- `internal/fabricengine#rejectionRetryKind`
- `internal/fabricengine#pushJob.awaitOrigin`

**Extends:**
- `internal/fabriccore/doc.go`

**Uses:**
- `internal/fabricengine#runRule`
- `internal/fabricengine#refuseUnsuppliedSeams`
- `internal/fabricengine#SetRunRecordPaths`
- `plan:internal/fabriccore#IsOwedClass`
- `internal/testkit/fabrickit#NewSnapshot`
- `internal/fabricengine#TestBoltReplay_NamesNoBoardFile`
- `internal/fabricengine#TestRunRule`

**Intent:**
`pushClassified`, the push every fabric push path runs per side, becomes the engine loop over a core rule (D2, D3).
Every decision it makes today, the transport retry, the origin lookup and its retry, the rejection settle loop and the failure class, moves to `fabriccore.PushSide` and `fabriccore.JudgePush`;
the engine keeps the git reads, the push, the error-to-class mapping and the failed-push record write.

Snapshot and phases:
- `Snapshot` gains `Push PushSideFacts` and `Backoff []time.Duration`, today's `gitrepo.PushBackoffDelays()`, which the engine sets before the first answer.
- `PushSideFacts` holds what `pushClassified` reads today, in fabric's vocabulary:
  side, branch (empty when it cannot be read), prime, the job options the rules read (`SingleAttempt`, `AfterRecords`, ceiling, landings, `OwedUnrecorded`, run slug),
  whether the namespace read failed and its error,
  the upstream state (none, set with a pruned tracking ref, tracked),
  whether the side had unpushed commits and whether that read failed,
  origin's view (head, gone, ended, unchecked),
  the rewind verdict and its dropped commits (`classifyRewrite`'s answer),
  the unmade commit and origin tip a refused reset named,
  the board facts `refuseUnseededBoard` reads today,
  and the judgment facts `failRejection` reads: whether the fetch failed, origin's tip, and whether origin holds commits the branch lacks.
- `pushJob.read` fills `FieldPushSide` in `PhaseUnderLock`, and the origin view in `PhaseOrigin` only when the rule asks for it, which it does only for a weft or task branch with an upstream, as `checkOrigin` reads it today;
  the board facts are read only for a Bolt side in a one-repository records store with no upstream, as today.
  `FieldPushJudgment` is read only after a rejection the retries did not settle, as `failRejection` fetches today.
  So no path starts more git processes than today (D2).

Actions and outcomes:
- `PushSide` answers `ActionPush` for the side's push, `ActionWait` with the next backoff delay, and `ActionSeamCall` with `SeamRejectionRetry` for the retry of a rejection, whose runner is today's retry function (`retryWeftPush`, `retryBoltPush` or `repushOverOriginContained`).
  Each list ends at an action whose answer it branches on (D3).
- The engine keeps each action's error beside its outcome, by the outcome's index.
  The verdict names a class and the outcome it rests on, and `pushClassified` builds the record and the returned error from that kept error, as today:
  an `*AnchorStopError`'s way forward, a `*replayConflict`'s paths, and a takeover error returned as the same object,
  so `errors.As` for `*TakeoverError` and `*TakeoversError` and `errors.Is` for `ErrTakeoverPassOwed` and `ErrHandedOver` hold for every caller (`internal/pairteardown/handover.go`).
- `outcomeClassOf` (`executor.go`) takes over `rejectionRetryKind`'s mapping, in its order:
  an `*AnchorStopError` is anchor stop, auth refused is auth refused, a takeover stop is `OutcomeTakeover`,
  a protected-branch refusal that wraps `gitrepo.ErrPushRejected` is `OutcomeProtected`,
  and one that does not, such as a hook's or ruleset's refusal of `resetUnnamedTip`'s leased reset, which `gitrepo.UpdateRemoteBranchLeased` returns as a wrapped git error, is `OutcomeProtectedUnrejected`;
  a bare rejection is rejected, `ErrReplayConflict` is `OutcomeReplayConflict`, a destructive refusal is destructive refusal,
  `gitrepo.ErrPushRetriesExhausted` is `OutcomeRetriesExhausted`, ahead of the transport case, a transport failure is transport failure, `errOriginAhead` is `OutcomeOriginAhead`, and anything else is failed.
  A retry that reported a deferral returns `errRetryDeferred`, which `outcomeClassOf` maps to `OutcomeDeferred`.
  `OutcomeRetriesExhausted` is a push error no snapshot read returns, so `TestRunRule`'s read rows stand as they are.
  The mapping is the engine's one rule, as D3 keeps it; `TestOutcomeClassOf` gains a row per new class, `errRetryDeferred`'s and `OutcomeRetriesExhausted`'s included,
  a row for a protected refusal that wraps no `gitrepo.ErrPushRejected` mapping to `OutcomeProtectedUnrejected`,
  and the takeover and protected precedence over rejected and the exhausted precedence over transport.
- `coalescePushSides` runs `refuseUnsuppliedSeams(fabriccore.SeamRunRecordPaths)` first, before it takes the push lock, and returns the refusal unrecorded (D13).
  So a records push whose rejection retry reads the takeover (`ReadUpstreamTakeover`) never meets an unset seam mid-flow.

The rules `PushSide` and `JudgePush` carry, each as today:
- A transport failure of the push is pushed again after each backoff delay, unless `SingleAttempt`; the last transport failure stands once the delays are spent.
  An exhausted push (`OutcomeRetriesExhausted`) is never pushed again, as `retryTransportPush` re-pushes only what `gitexec.IsTransportFailure` classes as transport today.
- The origin lookup retries a gone branch whose entry is not ended, and a transport read failure, once per delay; a failure for any other reason reads unchecked and the push goes on.
  `SingleAttempt` returns a transport read failure at once.
  An ended entry answers a local `ActionRefDelete` of the branch's tracking ref, keeping the upstream setting, run as today through `DeleteRemoteTrackingRef` and not through `destroy.go`, which guards no tracking ref today;
  the verdict is no push and no record.
  A failed delete returns today's error, "delete the tracking ref of the branch …, whose entry origin ended", with no record.
  `goneEntryEnded` stays in `pushclassified.go`, called from `pushJob.read`, so `TestBoltReplay_NamesNoBoardFile`'s readers map stays true.
  A gone, not ended branch is a `gone` record wrapping `ErrBranchGone`; a rewound task branch, a `rewound` record holding the dropped commits.
- A rejection runs the retry seam call; it is called again after each delay while it ends rejected or failed, at most once per delay plus the first call, so no loop runs more rounds than today.
  A first push that is a protected refusal is a rejection, so it runs the retry seam call as today, where `mapPlainPushError` wraps a declining hook's refusal in `ErrPushRejected`.
  A retry outcome that is a protected refusal, of either protected class, stops the loop at once, as `rejectionRetryKind`'s not-retryable answer does today.
  An `OutcomeProtectedUnrejected` outcome's error is not joined to the rejection, as `settleRejection` returns a named class's error unjoined today.
  Origin ahead stops at once and keeps the first rejection; any other named class stops at once.
  An unnamed final failure is joined to the rejection.
- The verdict's class: anchor stop is `anchor_change` with its way forward; auth is `auth`;
  an `OutcomeProtectedUnrejected` outcome is `other` with that outcome's error and asks no `FieldPushJudgment`,
  as today `classifyFailure` names no class for it and `pushClassified` records it as `other`;
  a rejection the retries did not settle, an `OutcomeProtected` refusal of the push among them, reads the judgment facts, as `failRejection` does today:
  a refused reset's unmade commit is `not_fast_forward` naming it and origin's tip;
  a failed judgment fetch or tip read is `other` with that error joined;
  origin holding commits the branch lacks is `not_fast_forward` with the local commits; otherwise `other`, "a fetch found origin holding nothing this branch lacks".
  Replay conflict and destructive refusal are `not_fast_forward` with the conflicting paths and local commits; transport and an exhausted push are `transport`; failed is `other`.
  An `other` record of a push that carried landings and ended in a protected refusal, of either protected class, keeps `protectedParentWayForward`, as `pushJob.record` sets it today.
  A takeover is returned unrecorded; a deferral writes the deferred record and returns nil, or its unrecorded `*PushFailedError` under `OwedUnrecorded`.
  A namespace read failure is `other` before origin is read.
  A landed push records today's mutation entries and clears the side's record.
- `UnseededBoardRefusal` is `refuseUnseededBoard`'s refusal over the board facts, with today's four ways forward word for word, returned under the push lock its caller holds, as today, and with no record.

`pushClassified` keeps its signature and its `*PushFailedError` returns; it builds the job, sets `Backoff` and runs `runRule(fabriccore.PushSide, …)` with `pushJob.read`, the job's runners and `opts.BackoffSleep` as the sleep, then writes the record `JudgePush` names through `pushJob.record`.
The executor records in `fabricengine.Mutations` exactly the entries today's code records for a landed push, `branch_pushed` through `recordPushIfAdvanced` or `appendBranchPushed`, and none for a wait or a seam call (D4).
So `PATTERN-fabric-boundary`'s background, `### Destruction chokepoint`'s last bullet, is restated as D4 words it:
"The executor records in `fabricengine.Mutations` the entries today's code records for the same effect, kind for kind, and destructive primitives stay in `destroy.go`."
The retry functions' bodies, the weft and Bolt replays, stay one seam-call action each; their replay rules are the replay concern's (overview, Shared Decisions).
The executor section of `internal/fabricengine/doc.go` has its outcome-class sentence rewritten to list the classes `outcomeClassOf` maps after this card.

Tests (the commit message lists them):
- `TestPushSide` (core, tier0) over `fabrickit` snapshots with `Outcomes` and `ReadFailures` set per row:
  the transport loop pushes once per delay and stops after the last;
  an exhausted push is pushed once and records `transport`;
  `SingleAttempt` pushes once;
  the rejection loop calls the retry once plus once per delay over rejected outcomes and then stops with the rejection, the acceptance row of D3;
  a protected first push runs one retry call and then the judgment read;
  a protected rejection with origin holding commits the branch lacks reads `not_fast_forward`;
  an `OutcomeProtectedUnrejected` retry outcome stops the loop and records `other` with its own error, unjoined to the rejection, with no judgment read;
  a failed retry outcome keeps retrying and joins the rejection;
  origin ahead and each named class stop at once;
  a deferred retry outcome answers the deferred verdict;
  the origin lookup retries a gone, not ended branch and a transport read failure once per delay, answers the tracking-ref delete for an ended branch and unchecked for any other read failure;
  a rewound task branch answers the rewound verdict;
  a branch name that cannot be read pushes at once with no origin read;
  a Bolt side with nothing unpushed clears its record and ends;
  a branch with no upstream pushes with no origin read, and any other upstream read error is returned;
  a non-transport error while judging a gone entry is returned;
  `SingleAttempt` returns an origin-lookup transport failure at once;
  the prime, a pruned tracking ref and a branch not rewound proceed past the rewind check;
  a failed tracking-ref delete returns its error;
  and `JudgePush` rows for every class mapping above, the unseeded-board refusals and the namespace failure.
  It adds about 2 seconds to one tier0 run.
- `TestPushFailureClasses` keeps its rows that pin a real remote's answer or an interleaving:
  `APushThatLostARaceIsRetriedUntilItLandsWithNoRecord`, `AJudgmentFetchThatFailsAfterTheFinalRejectionIsRecordedAsOtherWithTheFetchError`, `ACodeRejectionWhoseRetryIsRefusedForAuthIsNotRetriedAgain`,
  `APushOfABranchNeverPushedThatFailsInTransitLandsOnTheRetryOnceOriginAnswers`, `AProtectedBranchRefusalRecordsTheRejectionClassAndIsNotRetried`, `APushToAnUnreachableOriginWritesATransportRecord`,
  `AWeftPushDefersUnderTheWriteLockAndTheNextCommitsPushLandsIt`, `AWeftReplayMeetingABothSidesStatusChangeWritesANotFastForwardRecordAndMails`, `AWeftPushThatMustReplayStopsOnAnAnchorChange`,
  `ATaskBranchRewoundOnOriginIsHeldBackUntilTheRewindIsSettled`, `RepeatedAuthRefusalsShareOneRecordAndOneMailAndALandedPushRemovesIt` and `APairPushFromARecordsStoreWithNoNamespacedFetchRefspecRecordsTheRestoreCommands`, which pin the git reads and actions the rule's verdicts rest on.
  It keeps `ARejectionOfABranchWithNoUpstreamWhoseNameOriginAlreadyHoldsIsADivergence` and the `Code` and `RecordsWithNoUpstream` rows of `ARejectionThatOutlastsTheBackoffWithOriginHoldingNothingTheBranchLacksIsNoDivergence`, which pin `failRejection`'s read of origin's same-named branch,
  and `ThePrimesCodeBranchPushedAfterAnOriginRewindIsPushedAndRecordsNothing`, which pins the prime read that skips the rewind check.
  It deletes that test's `Records` and `Bolt` rows, `APushFailingForNoNamedReasonWritesAnOtherRecordAndMails` and `APrimeCodePushIsRecordedWithThePrimeFormOfItsWayForward`, whose verdicts are `TestPushSide` rows and whose way-forward texts are card 5's.
  Its rows that rest on the board's replay move in card 12.
  The one destructive action of this concern, `resetUnnamedTip`'s leased reset of origin's code branch, keeps its lease at execution in `destroy.go`'s `updateRemoteBranch` (D4).
- `TestRejectedCodePush_RefusesACommitThisMachineDidNotMake` stays as it is; it pins the reset's refusal over a commit this machine did not make.
  D4's tier1 row of a destructive action whose target turned dirty after the snapshot needs an action whose `destroy.go` request checks dirtiness.
  This concern's one destructive action, `resetUnnamedTip`'s leased reset, passes `dirtinessNA` (`pushclassified.go`), and no other action of the concern goes through `destroy.go`,
  so that row lands with the first part whose concern moves a dirtiness-checked action, the teardown part (D9); the live-pair half of the criterion is card 11's row.
- `TestPushClassified_Snapshot` is a tier1 test in package `fabricengine_test`, the push flow's seam test (D6);
  package `fabricengine` cannot hold it, since `internal/hubforge` imports `internal/fabricengine`.
  It reads the snapshot through `PushClassifiedSnapshotForTest`, an `export_test.go` accessor that runs `pushClassified`'s `runRule` and returns the snapshot its phases read.
  On a hubforge pair it pushes the records side with one unpushed commit, asserts that snapshot (the push side's facts and the phases in order) and then the end state, the branch on origin and no record.
  A second row, a code branch with no upstream, asserts the snapshot holds no `PhaseOrigin` and no judgment read, so that path reads no more than today (D2).
  Card 8's records-landing loop runs the same reads per push and shares it.
  The per-verb reader makes today's git and `gitrepo` calls unchanged, so its handling of a ref storage other than `files` is today's.
- `TestCoalescePushBothAt_AdvancesBothSidesAndLeavesNoWarpRootLock` asserts the push's exact printed `mutations` array against the array today's code prints (D4, D5);
  `CoalescePushRequest` runs the same `coalescePushSides` and shares it.
- `TestCoalescePushRequest_CoveredCallDefers`, all three of whose subtests call `CoalescePushRequest`, and `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock` push plain repos with bare origins and build no hub,
  so each now meets the pre-lock refusal.
  Each sets the run-records paths seam itself through `SetRunRecordPaths`, with paths under its own temp repos, restores it before it returns and stays free of `t.Parallel` (D13);
  package `fabricengine` cannot take `hubforge.InstallRequiredSeams`, since `internal/hubforge` imports `internal/fabricengine`.
  Their assertions are unchanged.
- `TestInstallRequiredSeams` gains a row: with the run-records paths seam unset through `fabricengine.SetRunRecordPaths(nil)`, `CoalescePushRequest` of a pair with an unpushed records commit refuses naming the seam, and origin's records branch is unchanged.

**ImpactSummary:** pushClassified keeps its signature, so its callers are untouched; coalescePushSides gains a pre-lock seam refusal; the executor's class mapping gains the new classes; one PATTERN background clause is restated.
