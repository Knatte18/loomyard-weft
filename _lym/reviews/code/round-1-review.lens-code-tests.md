```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails if it breaks.

**Behaviour → test that pins it**

1. Session spec (interactive, unawaited, deny-publish, role `pr-reviewer`, skills, model choice, prompt carries the URL, pane registered, stopped on a merged verdict) → `TestPRReview_Session/AnOpenedPullRequestStartsOneInteractiveSessionAndAMergedVerdictStopsIt`.
2. The session's identity is written to the record → `TestPRReview_Session/TheStartedSessionsIdentityIsRecorded`.
3. A live recorded strand is reused → `ARecordedLiveStrandIsReusedWithNoStart`.
4. A dead strand is resumed by its session id → `ARecordedDeadStrandIsResumedBySessionID`.
5. A recorded closed pull request gets a session, and the Stuck verdict stops it → `AStuckVerdictStopsTheSession`.
6. A merged pull request and an already-ended review get no session → none.
   No row asserts zero starts on the merged re-entry path or the `ReviewEnded` path.
7. A cancelled context leaves the session running → `ACancelledContextLeavesTheSessionRunning`.
8. A session that ends while the review is open halts naming `lym loom resume` → `ASessionNoLongerLiveWhileTheReviewIsOpenHaltsNamingLoomResume`.
9. A failed liveness read logs one Warn and the wait goes on (`liveWarned`) → none.
10. A non-transient start failure halts; a transient one is an error → `ANonTransientStartFailureHaltsNamingItAndLoomResume` and `ATransientStartFailureIsAnError`.
11. A start failure inside `resumeSession` (from the pause hold or from the wake) is turned into a halt → none.
12. A pause stops the session and its end resumes it → `APauseStopsTheLiveSessionAndTheEndOfThePauseResumesIt`.
13. No pull request: the session is stopped before the record is dropped → `NoPullRequestStopsTheRecordedSessionBeforeDroppingTheRecord`.
14. Stop-before-read in `wake`: the tree is read again after the stop, and the session is resumed when the read leaves the review open → none.
    The only stop row is the merged verdict. No row has the session stopped, the review kept open and the session resumed. No row has the tree changing between the first read and the re-read.
15. `sessionStopWanted`'s Continue and approval branches → none.
    The wait tests' fake reads every strand as live and asserts no stops, so deleting either branch passes every test.
16. The `ensureSession` record write fails → the started strand is stopped → none.
17. A failed `RegisterSession` is only logged → none (`registerErr` is never set by any test).
18. A gate failure is mailed once per head, `MailedHead` is recorded, and the head is not gated again → `TestPRReview_Wait/AFailingGateOnAHandCommitMailsTheSessionOncePerHeadAndPushesNothing`.
19. A failed mail is logged, and an empty strand name is logged instead of mailed → none.
20. A runner crash halts and mails nothing → `ARunnerCrashHaltsTheRowNamingTheCrashAndMailsNothing`.
21. `MailedHead` cleared by pr-continue makes the next wake gate that head again → none in landingshed.
    Only the verb's record write is pinned (row 23); no wait row starts from a cleared mailed head and checks the gate reruns.
22. pr-continue admits pr-reviewer and orch strands, numbered forms included → `TestPRContinueVerb/APRReviewOrOrchStrandSetsContinue`.
    It refuses another role → `ACallerWithAnotherStrandNameIsRefusedAndWritesNothing` and `…BeforeTheRecordIsRead`.
    A strand that does not parse is refused → none. No row covers the `agentname.Parse` error branch, nor a two-part `<short>:pr-reviewer` name.
23. pr-continue clears `MailedHead` when Continue is already set → `AMailedHeadIsClearedAndContinueStaysSet`.
24. pr-continue's help audience is now `AudienceRole` → none found.
25. `landing.pr_reviewer`: template default, model-spec validation, local layer wins → `TestLoadConfig_TemplateDefaults`, the `TestLoadConfig_InvalidKeys` row "malformed pr_reviewer", and "a local pr_reviewer wins over the hub file's".
26. The `pr-reviewer` role is listed in agentconfig with its template skills → `TestLoad_TemplateListsEveryRole` (generic, `internal/agentconfig/config_test.go:70`).
27. The reinterpretation is dropped from configreg → no test needed (it is a removal).
28. The PR-Review opening stencil renders the parent and edit directives → `contracts/stencils/parentdirective_test.go:128,146` over `roleOpeningStencils`.
29. Engine parity of the PR-Review spec → the `buildPRReview` producer row in `TestProducers_CodexManifestHoldsClaudes`.
30. `wireLanding` fills `Landing.PullRequest.SessionLive` and `RegisterSession` → none.
    `TestLandingDeps_EveryFieldPopulated` exempts both, `wiring_seams_test` lists them as intentionally nil, and `TestWireLanding_FillsSamplerSessionSeams` checks only the sampler seams.
31. `pullRequestDepsGuardedBy` fills `MailAgent` and `Sessions` → `TestLandingDeps_EveryFieldPopulated` (it checks non-zero only).

**Findings**

- **MEDIUM**, design, `internal/landingshed/prreview.go:597-618`. Stop-before-read in `wake`, the tree re-read and the resume after a stop that leaves the review open have no test row. Approval at the gated head with a dirty or moved tree, and Continue with stale text, are unpinned. The wait tests' fake answers every strand live and no wait row asserts stops, so dropping `sessionStopWanted` or the resume passes. Evidence: `prreview_session_test.go` rows (the only stop row is the merged verdict, line 679); `prreview_wait_test.go` has no stop assertion (grep of lines 44–466).
- **MEDIUM**, design, `internal/loomcli/sharedbootstrap.go:434-440`. No test pins that `wireLanding` fills `Landing.PullRequest.SessionLive` and `RegisterSession`, which `checkWiring` requires before PR-Review can run. Evidence: the `landingdeps_test.go:152-157` exemption, the `wiring_seams_test.go` intentionally-nil entries, and `wiring_publish_integration_test.go:60-103`, which covers the sampler only.
- **LOW**, design, `internal/landingshed/prsession.go:85-96`. The record write failing after a start, which must stop the strand to prevent a second session on re-entry, has no row.
- **LOW**, design, `internal/landingshed/prreview.go:727-735`. `mailFailure`'s failed-post and empty-strand branches have no row, though `MailedHead` is recorded on both. Evidence: `prreview_wait_test.go:84-90`, where the mail seam never fails.
- **LOW**, design, `internal/landingshed/prsession.go:181-188` and `prreview.go:420-423`. A start failure inside `resumeSession` (from `holdWhileCovered` or from the wake) reaching `Call`'s halt conversion has no row.
- **LOW**, design, `internal/landingshed/prreview.go:265-272`. The rule that a merged recorded pull request and a `ReviewEnded` re-entry start no session has no row asserting zero starts.
- **LOW**, design, `internal/landingshed/prreview.go:626-637`. The `liveWarned` path, where a failed liveness read lets the wait go on, has no row; no test sets `liveErr`.
- **LOW**, design, `internal/loomcli/prcontinue.go:247-256`. The refusal of an unparsable strand by `prContinueAdmitsStrand` has no row. Evidence: `commitrecords_test.go:147-205`.
- **LOW**, design, `internal/landingshed/prreview.go:565`. No wait row covers a cleared `MailedHead` making the next wake gate the same head again, the retry half of the pr-continue contract.
- **NIT**, design, `internal/loomcli/prcontinue.go:95`. The audience change to `AudienceRole` has no test I found.

Tool calls:
1. Grepped the wait and Call tests, the agentconfig, stencils and configreg tests, and the loomcli tests for session, mail and admission assertions — I had not read the test bodies beyond the diff.
2. Grepped the loomcli tests for `wireLanding` and the seam names, plus the agentconfig test names and the stencil role tests — to find coverage of the wiring and the role list.
3. Grepped `wiring_darn_test.go` and `wiring_publish_integration_test.go` for the PR-Review seams — to confirm whether any wiring test asserts them.
