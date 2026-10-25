# Card 4 — drop-review-verbs

**Edit:**
- `internal/loomcli/review.go`
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli#inReviewGroup`
- `internal/loomcli#Command`
- `internal/loomcli#TestShuttleWaiting`
- `internal/loomcli#TestVerifyWaiting`
- `internal/battencli#battenCLI.wire`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.runningStep`
- `contracts/specs/loom-status-spec.md`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#newReviewStore`
- `internal/loomcli#openReviewRequest`
- `internal/loomcli#writeReviewFile`
- `internal/loomcli#assertReviewRefusal`
- `internal/loomcli#testStatusWay`
- `internal/loomcli#TestReviewVerbs_Success`
- `internal/loomcli#reviewVerbCalls`
- `internal/loomcli#TestReviewVerbs_NoOpenRequest`
- `internal/loomcli#TestReviewVerbs_VerdictAlreadyRecorded`
- `internal/loomcli#TestReviewVerbs_Expired`
- `internal/loomcli#TestReviewVerbs_ReviewFileMissingOrEmpty`
- `internal/loomcli#TestReviewSlugArg`
- `internal/loomcli#reviewErrMsg`
- `internal/loomcli#rejectCappedRound`
- `internal/loomcli#fakeRunStatus`
- `internal/loomcli#TestReviewApprove_SupersedesCapRejectOnBlockedRun`
- `internal/loomcli#TestReviewApprove_SupersedeRefusals`
- `internal/loomcli#reviewStore`
- `internal/loomcli#openReview`
- `internal/loomcli#waitingNote`
- `internal/loomcli#TestReviewWaiting`
- `internal/loomcli#reviewAsNext`
- `internal/battenshed#failReviewWaitNote`
- `internal/battenshed#innerRunProducer.reviewWaitNote`
- `internal/battenshed#TestInnerRun_RunningArmReviewWaitNote`
- `internal/loomcli#TestResolvePersistentPreRun_SkipsReviewGroup`

**Create:**
- `internal/loomcli#TestResolvePersistentPreRun_SkipsSlugGroups`

**Intent:** D2: the `lyx loom review` verb tree (`notify`, `delivered`, `approve`, `reject`), `lyx loom status`'s parent-review waiting note and batten's `ReviewWait` seam go.

- `review.go` keeps only what the `circling` and `decision` groups share: `reviewWaySlugForm`, `reviewTargetDeps`, `defaultReviewTargetDeps`, `resolveReviewTarget` and `reviewStatusReader`.
  Every verb body, `reviewStoreFor`, `reviewStatusWay`, `reviewStoreErr`, `reviewSlugArg`, `reviewState`, the `review` way-forward constants and `reviewCmd` go, with the `internal/parentreview` import.
  The file header now says it holds the target resolver the slug-taking loom groups share, and keeps the reason their pre-run is skipped (arm never runs for a group and never reads a slug as a run-id, which holds under `cobra.EnableTraverseRunHooks`), since `circling.go`, `decision.go` and `peers.go` point at it.
  `resolveReviewTarget`'s doc names the groups that call it, `circling` and `decision`.
- `Command` registers no `review` subtree, and `inReviewGroup` skips only `circling`, `decision` and `peers`.
  `TestResolvePersistentPreRun_SkipsReviewGroup` looks the removed `review` group up and fails without it, so it goes, replaced by `TestResolvePersistentPreRun_SkipsSlugGroups`: it keeps the old test's pre-run skip assertion over the `circling`, `decision` and `peers` groups, and asserts the `loom` group has no `review` child, so `lyx loom review` is no command.
- `reviewstatus.go`: `reviewWaiting` and `reviewWaitPrefix` go, and `reviewWaitingFor`'s chain ends in a step that reports no wait, so a gate wait, a parent lock wait, a verify and a shuttle wait are reported as before; its header and doc drop the reviewer.
  `TestShuttleWaiting` and `TestVerifyWaiting` give the chain a fake next step returning a fixed note in place of the review store, keeping every row's assertion on which wait wins.
- Batten: `InnerRunDeps` loses `ReviewWait`, `childWait` its `reviewNote`, and `runningStep` its reviewer-wait reason, so a running child's reason is always the plain wait line; `battenCLI.wire` stops building the seam over the parent-review store.
- Docs: `loom-status-spec.md` drops the parent-review wait from the waiting order and its `parent review: ` note form; `docs/overview.md` drops `review notify|delivered|approve|reject` from loom's verb list and `internal/parentreview` from loom's package list, and its `status` waiting-note sentence names a verify and a shuttle wait, no longer Discussion-Write's parent-review gate.

A run halted `blocked` at the old parent-review reject cap resumes with `lyx loom resume`, and its stored stop text still names the removed `lyx loom review approve`, as D2 records.

**ImpactSummary:** Removes one loom verb group, one status note and one batten seam; callers are local to loomcli, battencli and battenshed.
