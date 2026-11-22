```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none (lens code-tests).

**Per behaviour: pinning test**

- `beam.yaml` `autonomy` key, lookup and local-value checks: pinned by `TestLookupTaskSettings` and `_RefusesAValueTheKeyDoesNotTake` (`dobbyengine/tasksetting_test.go`) and by the `TestPrepareRunSettings` rows (`configreg/runsettings_test.go`).
- `LoadConfig` refusing a non-boolean resolved `autonomy` (`config.go`, the `perTask` loop): **none**. The refusal rows go through `LookupAutonomy`, not `LoadConfig`.
- `autonomy: false` in the beam template: no visible row. A template test may exist outside the diff.
- Finalize's config-change notice leaving `autonomy` out: pinned by `TestFinalize_ConfigChangeNotice` (two rows).
- `hubgeom.Autonomy`: pinned by `TestAutonomy`: start at the named worktree, task over beam, the prime's name, and warn-once on a failed read.
- `NotifyParentHalt` and `NotifyRunFromHalt`: pinned by `TestNotifyHalt_CarriesTheRunsAutonomy`.
- Switching each halt-notice caller to the stamped variant: **none**. See F1.
- `ClearParentAsk`: pinned by `TestClearParentAsk`.
- `agreementGate` treating a cleared ask as no ask: pinned by a `TestParentAgreementGate` row.
- Drift judge halting on an ended ask, resume re-ask, kill mid-wait, lost snapshot, and the diff-command path: pinned by the `TestDriftVerify` autonomy block.
- `driftengine.ApplyAutonomy` and `AutonomyForcedSetting`: pinned by `TestApplyAutonomy`.
- `driftDeps` config seam applying autonomy: pinned by `TestRunAutonomy_ForcesTheRunsKeysOverARealHub`.
- Loom forcing, covering `wire`, `ParentReviewWanted`, `DiscussionTable` and the setting lookup: pinned by `TestApplyAutonomy` (loomengine), `TestAutonomySettingLookup` and the integration test above.
- `pr_review_parent_timeout_min` default and `&gt;= 1` check: pinned by `TestLoadConfig_Loads` and `TestLoadConfig_Refuses`.
- `pullrequest.Record.Autonomy` and `ClearReviewSession`: pinned by `TestRecord_RoundTripAndUpdate` and `TestClearReviewSession`.
- `PRReviewAsker.Ask` and `ReadPRReviewStanding`: pinned by `TestPRReviewAsker_Ask` and `TestReadPRReviewStanding`.
- Session recording autonomy once, non-interactive spec, ask at the head, recorded value outliving the layers: pinned by `TestPRReview_Session` rows.
- Autonomy prompt block versus operator block: pinned by the prompt-contains checks in the Session rows.
- Judge ending the review only on a satisfied standing, clearing the flag and mailing stay-open, closed pull request halting, approval bypass: pinned by `TestPRReview_Wait` rows.
- `judgeParentAnswer` failing because `PRReviewStanding` is nil or returns an error: **none**. See F3.
- `parent-review ask` verb refusals and success: pinned by `TestParentReviewAskVerb`.
- `parent-review ask` wiring (`prRecord`, `head`, `askPR` in `preRun`): **none**. See F4.
- Answer verb's PR-Review way forward: pinned by four `TestParentReviewAnswerVerb` rows.
- `pr-continue` under autonomy (standing refusals, mailed-head clear, closed refusal, satisfied sets continue), tested against the real standing reader: pinned by `TestPRContinueVerb` rows.
- `prContinueReviewOf` (head, standing and API closed reads) and the read-error branches: **none**. See F4.
- `Goto` and `Relaunch` calling `Admitted` after the move, and not on a refusal: pinned by `TestGoto_Guard` and `TestRelaunch`.
- `Admitted` returning an error, which the doc says comes back with the move already written: **none**. See F3.
- `prReviewEntryReset` and `clearPRReviewOnGoto`: pinned by `TestPRReviewEntryReset` and `TestClearPRReviewOnGoto`.
- The three `Admitted` wiring sites (`arm.go`, `arbiterconsume.go`, `drivewiring.go`): **none**. See F2.
- Beam-Await drain notice (post once, marker, re-entry, dobby-down, reopen, no commits, ready record, autonomy off, failed or unposted post, seam errors): pinned by `TestBeamAwait` rows.
- Drain wiring (`HoldsCommits` and `WorkRemains` in `pawlwiring.go`, covering drafts and child pairs): **none**. See F4.
- Slack hook filter: pinned by four `TestHook` rows.
- `orchAutonomous` wiring in `slackcli/wiring.go`: **none**. See F4.
- `WithLaunchFacts` and `ResumeContext`: pinned by `TestWithLaunchFacts` and `TestResumeContext_RendersPointerPerPhaseAndWritesMark`.
- `WithLaunchFacts` at the three watcher sites and the two `start.go` sites: pinned by the Watcher and `TestStart_ResumesTheRecordedSession` rows.
- `orchPaths` setting `paths.Autonomy`: **none**. `TestStart` sets `h.cli.paths.Autonomy` directly. See F4.
- Autonomy line in the orch role texts, glossary and help: prose, exempt.

**Findings**

- **F1, MEDIUM, design:** The halt-notice stamp is pinned only at the helper. No test fails if a caller goes back to the unstamped `NotifyParent` or `NotifyRunFrom`. The sites are `battencli/notices.go:27-30`, `loomcli/drivewiring.go:~258`, `loomcli/prepush.go:~79`, and any other halt caller the diff did not touch. Fix: one row per wired seam asserting `; autonomy:` in the posted line, or a table over the callers.
- **F2, MEDIUM, design:** Nothing pins the three places that pass the PR-Review reset into a goto. They are `loomcli/arm.go:479`, `loomcli/arbiterconsume.go:30` and `loomcli/drivewiring.go:290`. Removing any one fails no test, and the goto then keeps the old autonomy and session, breaking the `goto`-clears acceptance criterion. Fix: a test that each built spec or request has a non-nil `GotoAdmitted` or `Admitted`, or an integration row that runs `goto PR-Review` over a record.
- **F3, LOW, design:** Three failure paths have no row.
  - `LoadConfig` refusing a non-boolean `autonomy`.
  - `judgeParentAnswer` failing on a nil or erroring `PRReviewStanding`: `prreview.go` (`checkAutonomySeams` and the standing-read error).
  - `Admitted` returning an error from `Goto` or `Relaunch` (`shedengine/goto.go:~153` and `~291`).
  Fix: add one row each.
- **F4, LOW, design:** Production wiring closures have no test: `prContinueReviewOf` (head, standing, API closed read), `parent-review ask` `preRun` seams, the `AskPRReview` and `PRReviewStanding` closures in `landingdeps.go`, `prReviewAsker` and its use of `PRReviewParentTimeoutMin`, `HoldsCommits` and `WorkRemains` in `pawlwiring.go`, `orchAutonomous` in `slackcli/wiring.go`, and `orchPaths` `paths.Autonomy`. Their unit-level logic is covered, but a miswired closure would pass. `TestRunAutonomy_*` and the wiring-seam tests only check the seam is non-nil. Fix: add a hub-level row for at least the Beam-Await drain, which is the only wiring that decides a behaviour (drafts and child pairs).

Tool calls: none; my context already held every diff.
