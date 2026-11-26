```yaml
lens: code-tests
outcome: reported
```

The code-tests lens: for each behaviour the diff adds or changes, the test row that pins it, or none.

**Behaviour → pinning test**
1. `beamsetting` values `review|off|trail|merge-in` and `ResolvePullRequest` under the new names → `TestResolvePullRequest` (`internal/landingshed/pullrequestmode_test.go:25-30`). Pinned.
2. `dobbyengine.LoadConfig` loads `review`/`trail` and refuses `on`/`merge` with a way forward → `TestLoadConfig` (`internal/dobbyengine/config_test.go:39-42`). Pinned.
3. The per-task lookup refuses `on`/`merge` in a worktree or local file → `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake` (`internal/dobbyengine/tasksetting_test.go:595-596`). Pinned.
4. The template default is `review` → `TestLookupTaskSettings` "nothing set anywhere" (`tasksetting_test.go:556,567`). Pinned.
5. `ReinterpretPullRequestModes` rewrites the hub file, flags worktree and local files, keeps comments, leaves current values and `land_notice: on` alone → `TestReinterpretPullRequestModes` (`internal/dobbyengine/reinterpret_test.go:299-454`). Pinned.
6. `ReinterpretPerTaskSettings` maps `required` to `review`; the meaning text splits between `required` and the renamed values; the "worktree file decides" line moved out of `flagRetiredValues` → `TestReinterpretPerTaskSettings` (`reinterpret_test.go:247-285`). Pinned.
7. `CarryBeamTaskSettings` maps a renamed value in `run_settings` and in the retired entry key → `TestCarryBeamTaskSettings` rows `task-run-merge` and `task-legacy-on` (`reinterpret_test.go:465-466,495-496`). Pinned.
8. The beam module registers both reinterpretations in order → `TestLookup_BeamCarriesThePullRequestModesReinterpretationAfterThePerTaskOne` (`internal/configreg/configreg_test.go:49`). Pinned.
9. The board README mode legend → human-read prose, exempt.
10. A gate failure in the wait halts and pushes nothing → `AFailingGateOnAHandCommitHaltsNamingTheGateReasonAndPushesNothing` (`internal/landingshed/prreview_wait_test.go`, `wantReasonExact`). Pinned.
11. A runner crash halts → `ARunnerCrashHaltsTheRowNamingTheCrash`. Pinned.
12. The resume after a gate-failure halt "gates the task head again" (`prreview.go:517,539`) → **none**. No row re-enters `Call` with a record whose `GatedHead` is behind a clean task head.
13. `foreignHeadHalt`: a foreign head halts, an unreadable head halts, a lagging head waits, and a head equal to `GatedHead` passes → `AForeignGitHubHeadHalts…`, `AGitHubHeadTheTaskWorktreeCannotRead…`, `AGitHubHeadBehindTheTaskHeadKeepsWaiting`, and the `approvedAt("h1")` rows. Pinned.
14. The order inside `judge`: the foreign-head halt comes before the Continue ending and before the approval ending (`prreview.go:656-666`) → **none**. Every foreign-head row uses `seed("h1")` with no Continue and no approval.
15. A continue on an open PR at an ungated clean head gates it, then ends the review → `ContinueOnAnOpenPullRequestAtAnUngatedCleanHeadGatesItAndEndsTheReview`. Pinned.
16. Under no autonomy, a continue on a closed PR at the gated head ends the review → `ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt`. Pinned.
17. `Call` with no session, no ask and one commit per write → `TestPRReview_Call`, with `wantCommits` updated. Pinned.
18. Finalize drops its recorded-autonomy check on a closed PR → `YesWithAClosedRecordWithoutTheOperatorsEndingHalts…` and `TheOperatorsEndingOnAClosedPullRequestReopensIt…` (`finalize_pr_test.go`). Pinned.
19. `pr-continue` refuses a caller with a strand name, before the record is read and with nothing written → `TestPRContinueVerb` rows `ACallerWithAStrandName…` (`internal/loomcli/commitrecords_test.go:130-166`). Pinned at the verb.
20. The command reads `LYM_STRAND_NAME` through `os.Getenv(agentname.StrandNameEnv)` (`prcontinue.go:95`) → **none**. Every row passes the strand as an argument.
21. `pr-continue` succeeds without a write when the review already ended → `AnEndedReviewAnswersOkWithoutAWrite`. Pinned.
22. `AnswerParentAsk` has no unreached-tag exception → `TestAnswerParentAsk` rows for no-parent and undelivered, and `TestParentReviewAnswerVerb` "an undelivered ask is refused…". Pinned.
23. `ReviewInput.Full` is removed and a full generation comes only from `FullGeneration` → `ReviewInputStartsAFullGenerationWhenFullGenerationSays` (`bouncer_amendment_test.go`). Pinned.
24. `reviewInputs` ignores a `pr-findings` file → `TestReviewInputSeams` "ReviewInputsAnswerOnlyTheFixerReport…" (`subjects_test.go:171`). Pinned.
25. `pullrequest.Record` keeps an older record's dropped keys across a rewrite → `TestRecord_RoundTripAndUpdate` (`internal/pullrequest/record_test.go:121-140`). Pinned.
26. A retired `landing.pr_reviewer` in the hub file loads → `TestLoadConfig_TemplateDefaults` (`internal/landingshed/config_test.go:106`). Pinned.
27. A retired `landing.pr_reviewer` in a local layer is admitted, as commit 9752b18 says → **none**. `retiredLocalKeys` holds only `publish_verify` (`config_test.go:238-240`).
28. The retired `loom.pr` area and its flat names in the hub file are ignored → `TestLoadConfig_Loads` rows "retired pr area ignored" and "retired flat … ignored" (`internal/loomengine/config_test.go:358-365`). Pinned.
29. The retired `loom.pr` area in a local layer or in board `run_settings` is admitted → **none**. `autonomy_integration_test.go` dropped its `pr:` local line, and no row replaces it.
30. Autonomy no longer forces `pr.review_ask_parent` → `TestApplyAutonomy` and `TestAutonomySettingLookup` "APullRequestKeyIsNotForcedUnderAutonomy". Pinned.
31. PR-Review declares no `on_reroute` → `TestRecipe_PRReviewDeclaresNoReroute` and `TestRouting_ProgressAt`. Pinned.
32. Role `pr-reviewer` is removed, and `agents.yaml` refuses a hub that still lists it → **none** for that refusal of a removed role. This is pre-existing generic logic; noted for the handler only.

**Findings**

- **MEDIUM**, class `design`. Location: `internal/landingshed/prreview.go:165-245` (`openPullRequest`) with `gateHead` at `prreview.go:514-548`.
  Summary: the resume path after a gate-failure halt or a foreign-head halt has no test, and that is the path the new halt texts promise.
  Re-entering `Call` with a record whose `GatedHead` is behind a clean task head runs `pushTask` before the wait gates that head, and no row asserts the push order or count on that re-entry.
  Evidence: `prreview.go:171` (push at entry); `prreview.go:539,689` (halt texts); `prreview_wait_test.go` and `prreview_test.go`, where no row re-enters after these halts.

- **MEDIUM**, class `design`. Location: `internal/landingshed/prreview.go:656-666` (`judge`, `StateOpen` branch).
  Summary: no test pins that `foreignHeadHalt` runs before the Continue ending and before the approval ending.
  Moving that call below `if rec.Continue` would let a recorded Continue end a review whose GitHub branch holds a hand commit, and no row would fail.
  Evidence: `prreview_wait_test.go:431,446`. Both foreign-head rows use `seed("h1")` with no `withContinue` and no approval.

- **LOW**, class `design`. Location: `internal/landingshed/config.go:150-157` (retired `pr_reviewer`, CategoryNone).
  Summary: commit 9752b18 says a local layer now admits `landing.pr_reviewer`, but no test loads a local layer holding it.
  Evidence: `internal/landingshed/config_test.go:238-240`, where `retiredLocalKeys` holds only `publish_verify`; `git show 9752b1861 --stat`.

- **LOW**, class `design`. Location: `internal/loomengine/config.go:713-722` (`RetiredKeys` `areaPR` and its flat names).
  Summary: nothing tests that a task's local layer or `run_settings` carrying `loom.pr.*` still loads; the integration test lost its `pr:` local line with no replacement row.
  Evidence: `internal/loomcli/autonomy_integration_test.go:68`; `internal/loomengine/config_test.go:358-365` covers the hub file only.

- **LOW**, class `design`. Location: `internal/loomcli/prcontinue.go:95`.
  Summary: the command's read of `LYM_STRAND_NAME` has no test. Every row passes the strand to `prContinueVerb` directly, so passing `""` from the command would leave the operator-only refusal dead with every test green.
  Evidence: `internal/loomcli/commitrecords_test.go:141,160`.

Tool calls:
1. Grepped the tests for retired-key coverage and the foreign-head rows; my context lacked the full row list of `prreview_wait_test.go` and the config tests.
2. Grepped for local-layer tests of `review_ask_parent` and `pr_reviewer`, and ran `git show 9752b1861 --stat`, to confirm that commit added no local-layer row.
