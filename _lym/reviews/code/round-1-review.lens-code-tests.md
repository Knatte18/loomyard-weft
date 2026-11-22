```yaml
lens: code-tests
outcome: reported
```

Every behaviour the diff adds or changes, with the test row that pins it:

- `configengine.TopLevelValue` (highest present layer wins, unset key, a board-only read, an invalid layer): `TestTopLevelValue`, every row.
- The lookup's chain order (task local over task worktree file, task over parent, parent over prime, prime over hub default, hub default, template, no board dir): `TestLookupTaskSettings`, rows for both keys, plus its template and no-board subtests.
- The lookup refusing a bad value, naming the file and the way forward: `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`.
- `LookupLandNotice` returning on or off: `TestLookupTaskSettings` under the `land_notice` key.
- `hubgeom.TaskSettingChain` (task, beam, prime; an absent worktree left out; an empty slug; Bolt dir as the board): `TestTaskSettingChain`.
- The `hubgeom.TaskSettingChain` cycle guard ("a cycle among the parents ends the walk at the prime"): no test.
- `LoadConfig` validating the resolved `pull_request` and `land_notice` (merge-in and on load; required, all and per-task refused): `TestLoadConfig`, its new rows.
- `ReinterpretPerTaskSettings` (rewrites, comments kept, files left alone, prime files flagged): `TestReinterpretPerTaskSettings`.
- The beam module registering `beam-per-task-settings` in `configreg.Modules`, so the hub walk applies it: no test (finding 1).
- `pull_request` becoming `CategoryNone`, and a task's beam run settings preparing into the local layer: `TestModules_EveryTemplateKeyClassified` and the new `TestPrepareRunSettings` row.
- `landNotices` reading through the seam:
  - reading on or off: `TestLandNotices` rows `LandedNamesTheCommitOnce` and `ALookupReadingOffSendsNothing`;
  - a failed read keeps the last reading: `AFailedLookupKeepsTheLastReading`;
  - withdrawn and raised requests: `TheLookupFollowsARequest*`;
  - a failed board read keeps the membership: `AFailedBoardReadKeepsTheMembership`;
  - a member under a pull request gets its line whatever the lookup reads: `AMemberUnderAPullRequestGetsItsLineWhateverTheLookupReads`;
  - the beam with no pull request: `ABeamWithNoPullRequest*`.
- Posting until a post succeeds and dropping the memory: `AFailedPostIsOwedUntilItGoesOut` and `TestLandNotice_ReachesAnOrchStrandListedAfterTheLanding`.
- The view's `LandNotice` taken from the remembered reading: `TestBuildView`, row `ARunShowsItsRememberedLandNoticeRequest`.
- The overview rendering `land_notice` with no mode in the header: `TestDobbyLines`.
- The `hubDobbyDeps.LandNotice` seam reading the worktree layer: `TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer`.
- The `hubLandDeps.PullRequestMode` lookup from the beam's own worktree: the new assertion in `TestHubLandDeps_ReadyReads`.
- `judgeNoPullRequest`:
  - the mode-merge demand text "the beam's pull_request is merge": no assertion (finding 2);
  - required replaced by GitHub's rules: the `RulesRequired*` rows of `TestMarkReady`.
- `closeFollowUp` under merge and under rules: `TestPRClose`, rows `MergeReportsFinalizeLandsThroughAPullRequest` and `RulesRequired*`.
- `mergeParent` filing `beam.pull_request: merge-in` in `run_settings` and no board field: `TestMergeParent`.
- `foldLegacyBeamFields` and `setBeamSettingIfAbsent` (each fold, false dropped, an existing value wins, a non-mapping beam setting, a project's keys only dropped): `TestLoad`, the new rows.
- The upsert refusal of `pull_request` and `notify_on_land` with the `run_settings` hint: `TestUpsertKindAndRetiredKeys`.
- The run lock admitting a beam `run_settings` change: `TestRunLock`, row "beam run settings change".
- The render dropping the `pull_request` bullet: `TestRenderReadmeGolden`.
- `EndRaddleEntry` setting status only: `TestEndRaddleEntry`.
- `ResolvePullRequest` taking one mode (on, off, merge, merge-in; merge-in on a beam reads as off; rules; origin facts; an empty branch): `TestResolvePullRequest`, its subtests.
- The Finalize and PR-Review halt on a failed `Mode` lookup: the `AFailedModeLookup*` rows of `TestFinalize_PullRequest` and `TestPRReview_Call`.
- merge-in landing by a true merge: `AMergeInModeLandsByATrueMerge...`.
- `parentPullRequestMode` via the lookup (task's own value, a member reading its beam's, a beam reading its own or the prime's): `TestLandingDeps_PullRequestModeReadsThePerTaskLookup`.
- `DobbyTasks` failing on any unreadable parent of a released task: `TestDobbyTasks_AnUnreadableParentFailsOnlyAReleasedTask`.

Findings:

1. MEDIUM, design. `internal/configreg/configreg.go:142`: no test pins that the beam module carries the `beam-per-task-settings` reinterpretation. Dropping the `Reinterpretations` entry fails no test, so a hub's `required`, `all` or `per-task` would go unrewritten and stop beam.yaml loading. Evidence: `internal/hubreconcile/reinterpret_integration_test.go:78-85` registers only a fake slack reinterpretation, and no other `_test.go` names `PerTaskSettingsReinterpretationID`. Fix: assert in `configreg_test` or a hubreconcile walk test that the beam module applies `beam-per-task-settings` to a hub `beam.yaml` holding `required`.

2. LOW, design. `internal/beamcli/land.go:270-271`: the demand clause "the beam's pull_request is merge" that `judgeNoPullRequest` adds under merge is not asserted. Row `MergeWithTheSameMissingFactRefused` only checks `"no upstream"` and `"lym beam ready b"` (`internal/beamcli/land_test.go:537-545`). Fix: add the clause to that row's `wantErr`.

3. LOW, design. `internal/hubgeom/tasksetting.go:26-32`: the doc comment's cycle rule ("A cycle among the parents ends the walk at the prime") has no test row. Evidence: `TestTaskSettingChain` (`internal/hubgeom/tasksetting_integration_test.go:109-117`) covers a task, an absent worktree and an empty slug only. Fix: add a row whose origin records or board form a parent cycle, and assert the chain ends with the prime.

4. LOW, design (test economy). `internal/dobbyengine/tasksetting_test.go:135`: the report lists `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake` as contained. It pins the refusal naming the file that set the value (worktree, local or hub default), the value and the `lym config beam --set &lt;key&gt;=` way forward. Neither covering test asserts that: `TestLookupTaskSettings` asserts no error, and `TestLoadConfig` asserts only `LoadConfig`'s message. Fix: mark it `//testtiming:keep pins the lookup refusal naming the setting file, value and way forward` directly above its `func Test…` line.

5. LOW, design (test economy). `internal/boardengine/board_test.go:574`: the report lists `TestEntryDates` as contained by `TestRunLock`, `TestMakeSketch`, `TestPriorityRoundTrip`, `TestWriteSiblings`, `TestStoreDecisionRecord` and `TestRenderReadmeGolden`. It pins the created and revised dates each write stamps from the configured clock, which none of those asserts. Fix: mark it `//testtiming:keep pins the created/revised stamps of each write path`.

6. LOW, design (test economy). `internal/landingshed/prreview_session_test.go:502`: the report lists `TestPRReview_NoStopsTheRecordedSessionBeforeDroppingTheRecord` as contained by `TestPRReview_Call`. It pins that a no stops the recorded review session before the record is dropped, an ordering and side effect `TestPRReview_Call`'s rows do not read. Fix: mark it `//testtiming:keep pins the session stop before the record drop on a no`.

The mutation survivors at `landnotice.go:170`, `:173` and `:191` are log calls (`Fail` and `Recover`), and the one at `tasksetting.go:48` is a slice capacity, so none is a finding. The deleted rows and assertions across `boardengine`, `battencli`, `landingshed` and `dobbyengine` each pinned a behaviour the diff removes, so none is a loosened check.

Tool call: one grep over test files for a configreg test of the beam reinterpretation, the keep markers on the three contained tests, the pass harness's `LandNotice` default and the asserts of the `TestMarkReady` merge row. My context lacked those test files.
