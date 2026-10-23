# Card 42 — landing-drops-pr-producers

**Delete:**
- `internal/landingshed/publish.go`
- `internal/landingshed/prgate.go`
- `internal/landingshed/prgate_test.go`
- `internal/landingshed/pralign.go`
- `internal/landingshed/pralign_test.go`
- `internal/landingshed/pullrequest.go`
- `internal/landingshed/alignmentnote.go`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#parentGuard.moved`
- `internal/landingshed#holdParent`
- `internal/landingshed#rerouteIfMoved`
- `internal/landingshed#TestParentGuard_Moved`
- `internal/landingshed#TestGuardedRows_MovedParentReroutes`
- `internal/landingshed#TestNewPublish_Refusals`
- `internal/landingshed#TestPublish_NoPullRequestRequired_DoneWithoutPush`
- `internal/landingshed#TestPublish_StuckBeforePullRequest`
- `internal/landingshed#TestPublish_GitHubFailures`
- `internal/landingshed#TestPublish_NoExistingPR_CreatesAndLogsOnce`
- `internal/landingshed#TestPublish_OpenPullRequest`
- `internal/landingshed#TestPublish_ExistingClosedPullRequest`
- `internal/landingshed#TestPublish_MissingSummary_FailsLoudlyNoCreate`
- `internal/landingshed#TestPublish_CancellationAtEntry_SurfacesAsError`
- `internal/landingshed#TestPublishGitHubErrorReason_ClassifiesDistinctly`
- `internal/landingshed#TestFindPullRequest_QueryParametersAndEmptyList`
- `internal/landingshed#TestPublish_TransportPushFailure`
- `internal/landingshed#runAndGetReason`
- `internal/landingshed#requireReason`
- `internal/landingshed#publishGitHubServer`
- `internal/landingshed#newPublishGitHubServer`
- `internal/landingshed#publishGitHubServer.install`
- `internal/landingshed#jsonDecodeBody`
- `internal/landingshed#installFailingGitHubClientFactory`
- `internal/landingshed#publishVerifyFixture`
- `internal/landingshed#newPublishVerifyFixture`
- `internal/landingshed#publishVerifyFixture.call`
- `internal/landingshed#TestPublishVerify_Proceeds`
- `internal/landingshed#TestPublishVerify_HaltsOnVerifyResult`
- `internal/landingshed#TestPublishVerify_DirtyTreeHalts`
- `internal/landingshed#TestPublishVerify_HardErrorsAreReturned`
- `internal/landingshed#TestPublishVerify_NoPRRequiredRunsNoVerify`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/landingshed_test#publishIntegrationGitHubServer`
- `internal/landingshed_test#newPublishIntegrationGitHubServer`
- `internal/landingshed_test#publishIntegrationGitHubServer.install`
- `internal/landingshed_test#newPublishDepsAt`
- `internal/landingshed_test#TestPublish_MergesInCleanlyBeforeCreatingPullRequest`
- `internal/landingshed_test#TestPublish_RejectedPushNamesRemoteTipThenResumesAfterMerge`
- `internal/loomcli#landingAlignmentDeps`
- `internal/loomcli#alignmentNoteFile`
- `internal/loomcli#TestLandingAlignmentDeps_Config`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/shedrecipe#armPRAlign`
- `internal/shedrecipe#armPRGate`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#guardedRows`
- `internal/landingshed#TestGuardedRows_LockLifecycle`
- `internal/landingshed#TestGuardedRows_PausedLockWaitHalts`
- `internal/landingshed#failOnGitHubClient`
- `internal/landingshed#landingshedAllowedImports`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, recipe string, geom websterengine.Geometry, taskBranch, parentBranch string, pushSkipped bool, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, languageSkills []string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource, lock landingshed.ParentLockDeps) landingshed.Deps`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#verifySource`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#markDoneFixtureDir`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#intentionallyNil`
- `internal/shedrecipe#validLandingDeps`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#nilLegal`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Remove Publish, PR-Align, PR-Gate and the pull-request code from `internal/landingshed` (D13), now that no recipe names them.
Publish and its task-branch push, the pull-request lookup and client, PR-Align with the alignment note, and PR-Gate go with their tests; the shared test helpers the Merge-In and Finalize tests still use stay where they are.
`verifyGate.checkPublishVerify` and its `publishCommand` go, and so does the parent guard's moved-parent reroute (`moved`, `holdParent`, `rerouteIfMoved`), whose only callers were the removed rows and Finalize before card 27.
`Deps` drops `PushBranch`, `RemoteOnlyCommits`, `OriginURL`, `RunStart`, `ApprovalPath`, `RejectionPath`, `PublishCommand`, `PublishSkip` and `Alignment`, with the `AlignmentDeps`, `AlignmentConfig` and `AlignmentSession` types in the deleted files; the darn `PublishSkip` seam goes with it.
The package no longer imports the GitHub client, so its import allowlist test drops it.
loomcli's `landingDeps` drops its `originURL`, `pushBranch` and `runStart` arguments and their fields, `wireLanding` stops reading the origin URL, building the push and reading the run start for them, `verifySource` drops `publishCommand` and `publishSkip`, and the alignment wiring for PR-Align goes; every caller of `landingDeps` is listed.
The shedrecipe test fixtures and the test kit drop the removed seams.

`doc.go` drops the Publish, PR-Align, PR-Gate, pull-request and `PublishSkip` paragraphs and the claim that the deploy runs code PR-Gate reviewed.

**ImpactSummary:** Three landing producers, the pull-request code and nine Deps seams go; their remaining fillers are loomcli's landing assembly, the shedrecipe fixtures and the test kit, all listed.
