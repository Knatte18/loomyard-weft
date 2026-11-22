```yaml
lens: row-validity
outcome: reported
```

Task: row-validity lens review of the plan's planned test rows.

**Findings**

1. **MEDIUM. Card 6, D3 ended-ask halt.** The setup named for `TestDriftVerify_AutonomyHaltsOnAJudgeAskEndedUnanswered` cannot reach the new halt or the resume clear.
   - `TestDriftVerify_ParentAgreementCarriedWhileAsking` (`internal/driftengine/judgeparent_test.go:267-325`) calls only `parentAgreement` and `tableWithParent`. It never runs `DriftVerify.judge`, `runAgent`, `storeRulings` or `resumeJudgeSession`.
   - The one test that runs `runAgent` (`judgeparent_test.go:375-430`) ends with `errJudgeRunEnded` and a nil session. So `judge` returns before the planned check (`verify.go:453` onward: `if session == nil { return ... }`).
   - The "second call … leaves the ask `Cleared`" row needs `rec.Scope.JudgeSession.Open` to be true on re-entry. Only a run through `DriftVerify.Call` with records can produce that.
   - Fix: plan these as rows of `internal/driftengine#TestDriftVerify` (`verify_test.go:490`), which drives `Call`, and list that test and its seats fake on the card. Alternatively, name a harness that returns a session.

2. **MEDIUM. Card 4, D2 notice criterion.** No row reaches a halt notice. `TestWithRunAutonomy` tests only the helper.
   - The three post sites are `loomCLI.goDriverDepsFor`'s `HandBack.Notify`, `battencli#notifyRunParent` and `newPrePushCheck`'s notify. Nothing asserts that any of them appends the clause. A site left unwrapped passes every planned test.
   - Its rows repeat card 3's `TestAutonomy`: task `true` under beam `false`, and nothing set reads false. Both run over a second tier1 `hubforge` hub.
   - The "hand edit" row adds nothing either. `TestAutonomy` already writes each file directly, not through `--set`, and the reader caches nothing.
   - Fix: replace it with a row per post site, or at least one, that asserts the posted line ends with `; autonomy: <v>`. Keep at most one helper row.

3. **MEDIUM. Card 7, D3 Discussion criterion.** "The Discussion lead spawns non-interactive with autonomous mode text" has no row.
   - `TestRunAutonomy_ForcesTheRunsKeysOverARealHub` checks only `ParentReviewWanted` and `driftDeps(...).Config`.
   - `TestApplyAutonomy` checks the struct only. Nothing reaches `wire`'s `ApplyAutonomy` call before `DiscussionTable`.
   - The existing `internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape` (`wiring_test.go:542`), with `seedLoomConfigWithInteractive`, is where an autonomy row belongs.
   - The same gap holds for `wire`'s use of `autonomySettingLookup`. `TestAutonomySettingLookup` tests only the wrapper over a stub, so a `wire` that skips the wrapper passes.

4. **LOW. Card 14, D4 goto criterion.** `TestPRReviewResettingGuard` stubs `reset`. The real reset closure in `gotoGuardEndingFixers` is never run. That closure is `pullrequest.ClearReviewSession` over `shedrun.PullRequestRecord(...)` plus the commit.
   - A closure with a wrong record path, or none, passes.
   - The criterion "the session the re-entry starts … is new and interactive" is reached only by composition with card 8's `TestClearReviewSession`.

5. **LOW. Card 10, fixture wording.** The plan says `prReviewTestDeps` stubs `Autonomy` to read "a fixture field". `prReviewTestDeps(t, api)` returns a bare `Deps` and holds no fixture (`prreview_test.go:745-757`).
   - Rows must set `deps.Landing.PullRequest.Autonomy`, `AskPRReview` and `PRReviewStanding` per row, or `prReviewFixture` must gain the fields.
   - The card should say which, since `waitWorld` (card 11) builds on `prReviewTestDeps` too (`prreview_wait_test.go:67`).

6. **LOW. Card 11, unlisted fake fields.** `TestPRReview_Wait`'s new rows need things `waitWorld` does not hold (`prreview_wait_test.go:31-58`):
   - a scripted standing per row;
   - a count of standing reads, for the "approval ends Done with no standing read" row.
   - `waitWorld` and `newWaitWorld` sit under `**Uses:**`, not `**Edit:**`. Either list them under Edit with the new fields, or state that each row overrides `w.deps.Landing.PullRequest.PRReviewStanding` with a counting closure.

7. **LOW. Card 13, toggle row.** The row "recorded autonomy true with the layers now reading false" cannot fail if the code is wrong. `prContinueReview` has no layer or autonomy seam, so the stub has no "layers" input to set. The row repeats the autonomy-true rows. Cut it; the criterion holds by construction.

8. **LOW. Card 18, derived expectations.** The `TestStart_ResumesTheRecordedSession` and `TestWatcher_HandoffCompleteClearsThenResumes` assertions compare against `WithLaunchFacts(paths, …)`, the code under test.
   - A wrong `WithLaunchFacts` passes both. Only `TestWithLaunchFacts` pins the text.
   - Assert that the literal `autonomyOnLine` or `autonomyOffLine` suffix is present instead.

9. **LOW. Card 15, the new test pins old behaviour.** `TestReadBeamLandBlockers_CountsADraftMember` pins behaviour `ReadBeamLandBlockers` already has. `beamMembers` skips only `done` (`internal/battencli/beamland.go:107-140`), so a draft already counts.
   - The new `WorkRemains` wiring in `pawlDeps`, which this card adds, gets no row. That covers the error handling of `ReadBeamLandBlockers`'s `(BeamLandBlockers, error)` result, which the Intent's `!ReadBeamLandBlockers(prime, slug).Empty()` drops.
   - `HoldsCommits` is in the same position.

10. **LOW. Cards 1 and 3, test economy.**
    - Card 1 reruns all ten `TestLookupTaskSettings` rows under `AutonomyKey`. They exercise the same `lookupTaskSetting`, and only the boolean mapping is new behaviour.
    - Card 3's tier1 `TestAutonomy` rows repeat that precedence again. Its new behaviour is three things: the prime name becomes `""`, the spawn-free prime resolve, and failure reading false.
    - Fix: trim each to the rows that pin new behaviour.

11. **LOW. Card 16, unlisted harness.** `TestHook`'s new field needs the `slackCLI` harness in `cli_test.go` (around line 300-320) to set `orchAutonomous`. The harness builder is not listed on the card.

**Tool calls**
- Grepped the signatures of `driftDeps`, `pullRequestDepsGuardedBy`, `goDriverDepsFor` and `newPrePushCheck`, the named test functions, and the `waitWorld` struct. My context had only summaries of them.
- Read `internal/battencli/beamland.go` and grepped for its tests, to check draft counting and whether `ReadBeamLandBlockers` has an existing test.
- Grepped the package import directions (battencli/loomcli, hubgeom), to judge whether card 15's and card 16's wiring builds.
- Read `DriftVerify.judge`, `resumeJudgeSession` and `TestDriftVerify_ParentAgreementCarriedWhileAsking`, to check whether card 6's rows reach the halt.
- Read `TestDriftVerify_RunAgentCapsTheParentAskByTheRunsStart`, the list of drift test functions, the pause rows of the PR session tests and `prReviewTestDeps`, to check the harnesses cards 6 and 10 rely on.
