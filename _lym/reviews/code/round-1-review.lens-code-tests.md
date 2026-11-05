```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**
- Pawl rows, Finalize hold, retired map recipe shape: `TestPawlRecipe_HoldLiftsOnlyBeforeFinalize` (`internal/loomrecipe/pawlrecipe_test.go`) and the retired-map rows in `internal/shedbuild/parse_test.go`.
- Shed's retired-row move and pause, with unknown rows still refused: `TestStep_RetiredRowMovesAndPauses` (`internal/shedengine/retired_test.go`).
- `pawlengine.MoveOffRetired` side effects: `TestMoveOffRetired` (`internal/pawlengine/retiredmove_test.go`), which covers the carry cases, a resumed move, a failed commit, a failed notice, a repeat and a nil seam.
- Completed-wake repair reset: the `internal/sheddriver/drive_test.go` rows, `TestWakeStamp` (`internal/shedrun/wake_test.go`) and `TestEligibility_InterruptedPawlRows` (`internal/arbiterstop/arbiterstop_test.go`).
- Re-invoke interrupt policy rows: `internal/loomshed/interruptpolicy_test.go`.
- Beam-Await entry, accept and wake stamp: the `internal/pawlengine/beamawait_test.go` rows, including the goto, resume, failed ready read and stamp cases.
- Landing record: `TestLandingRecord` (`internal/pawlengine/landing_test.go`).
- `ReadMergeState`: `TestReadMergeState` (`internal/pullrequest/mergestate_test.go`), including the `has_hooks`, `unstable`, `dirty`, `behind`, merged and `unknown` retry rows.
- `RequiredChecks`: a subtest in `internal/pullrequest/client_test.go`.
- `ResolvePullRequest` beam branches: the `Beam` subtest in `internal/landingshed/pullrequestmode_test.go`.
- Finalize's recorded, scratch-record, required and `on`/`off` branches, and the refusal halts: the `internal/landingshed/finalize_pr_test.go` rows.
- Ready refusals 1 to 6: the `internal/beamcli/land_test.go` rows.
- `describe`, `pr open`, `pr update`, `pr status`, `pr close`: `TestDescribe`, `TestPROpen`, `TestPRUpdate`, `TestPRStatus`, `TestPRClose` (`internal/beamcli/describe_test.go` and `pr_test.go`).
- `parentRuleRequired`: `TestParentRuleRequired` (`internal/beamcli/pr_test.go`).
- Describe stamp and commit: `TestHubLandDeps_DescribeStampsAndCommits` (`internal/beamcli/deps_integration_test.go`).
- Fork precondition for a ready beam: the `internal/battencli/beamfork_test.go` rows and the integration subtest `MemberOfABeamFlaggedReadyIsRefused`.
- Dobby start hold and open-member notice once per episode: the `internal/dobbyengine/starts_test.go` subtests and the `view_test.go` row.
- Member-landing notice: `TestAnnounceLandings` (`internal/dobbyengine/announce_test.go`) and `TestLandingsPast` (`internal/beamcli/dobby_test.go`).
- `lym beam start` in landing: `TestOwnLandingHold` and the `beamUpRefusal` rows (`internal/orchcli/cli_test.go`), and `TestStart_OwnLandingStartsTheOrchAlone` (`internal/orchcli/start_test.go`).
- `BeamLineCommits`: `TestBeamLineCommits` (`internal/fabricengine/beamline_integration_test.go`).
- Orch role text: exempt as static prose.

**Findings**

1. MEDIUM, class design. `internal/beamcli/deps.go`, `hubLandDeps.ParentUnmerged`.
   - Behaviour with no pinning test: when the parent has an upstream and the push skip is unset, the real seam re-reads the parent's tips with the fetch under the remote-refs lock, then judges the upstream tip as well as the local one.
   - `TestHubLandDeps_ReadyReads` (`internal/beamcli/deps_integration_test.go:64-79`) covers only the local-tip case and the no-upstream case. The hub is built with `SkipPush: true`, and the test's own comment says unsetting the upstream fails there.
   - The `land_test.go` rows fake `ParentUnmerged`, so breaking the fetch branch or the upstream-tip ancestor check fails no row.
   - Decision record: D3 refusal 3, and the acceptance criterion on a parent tip not being an ancestor.

2. MEDIUM, class design. `internal/loomcli/pawlwiring.go`, `pawlRetiredMove` and `commitLandingDirectory`.
   - Behaviour with no pinning test: the production wiring of the retired move.
   - What it does: it ends the PR-Review session through `runproc.EndRole`, reads the scratch record, reads GitHub for `PullRequestOpen`, removes the scratch record, and commits the landing directory and the ready-record removal together through `CommitAnchoredPaths`.
   - `TestMoveOffRetired` drives `MoveOffRetired` with fakes. In `internal/loomcli`, `wiring_beam_test.go:95-99` asserts only `Pawl.RetiredRows`. A grep of `internal/loomcli/*_test.go` and `internal/sheddriver/drive_test.go` finds no test reading `RetiredMove` or `RetiredWayForward`. Nothing there shows that a new-binary shed step on a status naming `Parent-Align` runs the wired move and then leaves the run paused for the parent.
   - Breaking the wiring (a nil `RetiredMove`, or the commit pathspec or `EndRole` role) fails no row.
   - Decision record: D13 and its acceptance criterion.

3. LOW, class design, unconfirmed. `internal/loomcli/landingdeps.go`, `beamLandingRecordSeams`.
   - Behaviour: the real drop seam commits the landing directory after clearing the pull request.
   - The test diffs I listed show only 11 added lines in `internal/loomcli/landingdeps_test.go`, and I did not read them. The Finalize rows in `finalize_pr_test.go` use fake seams.
   - I could not confirm that any row exercises the real read, drop and commit seams, so the parent should check the 11 added lines before keeping this finding.

**Tool calls**
1. `git diff -U0 <base>..HEAD -- 'internal/*_test.go'`, filtered to test names and rows. My context held only test file names, not their contents.
2. A combined grep of `mergestate_test.go`, `deps_integration_test.go`, `finalize_pr_test.go` and the `loomcli` and `sheddriver` tests. I needed to confirm the `ParentUnmerged` upstream coverage, the `mergeable_state` rows, the drop-after-push assertions and the retired-move wiring coverage.
