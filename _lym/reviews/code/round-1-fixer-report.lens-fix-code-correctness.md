```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fixer's change set through the code-correctness lens.

Findings:

- **LOW**: `carryPullRequest` / `recordedPullRequest` in `internal/pawlengine/retiredmove.go` reads the landing record whenever the scratch record is absent, and does not check what GitHub reports for that pull request.
  - Input: a first move for a beam with no scratch record, whose landing record holds a pull request that GitHub has since closed or merged.
  - The new path returns that pull request without any state read, so `retiredMoveNotice` says "its pull request … stays open", which is false.
  - The review asked for this lookup only for a repeated move after `RemoveScratch` succeeded. The code cannot tell that case from a first move with no scratch record.
  - Impact is only the notice's wording. A real repeat after a failed commit is reported correctly, and the new subtest `a move repeated after a failed commit…` fails without the fix.

Other branches I ran step by step, with no defect found:
- **F1 `pullRequestLanding`, scratch open:** `pr.API` then `Status`, then an early `prLanding{merge, atOnce, api, record}`. This matches the old override's outcome, and the "Status 7, merge" calls of the existing open-scratch row are unchanged.
- **F1, scratch closed under `off`:** the resolver gives a no, and `dropRecordedPullRequest` reads Status, skips Close, removes the record and lands locally. The new row's calls `Status 7, RequiresPullRequest main, Status 7` match.
- **F1, scratch closed under `required`:** the resolver gives a beam yes marked `MergeAtOnce`, so it takes the reopen path with the calls the new row lists. With the old unconditional override, both new rows' `wantCalls` would differ, so they fail with the defect in place.
- **F1, failures:** a failed `pr.API` or `Status` goes through `githubFailure`, the same handling `landingForDecision` already applied.
- **F2 `BeamCommitsPast`:** the logic is a verbatim move of the old `MergedLacks` body, so `MergedLacks` behaves as before.
  - The `CommitsMissing` override in `landingDeps` is assigned before `PullRequest: pullRequest` is copied into `Deps`, and it ignores the task argument.
  - The only reader of `CommitsMissing` on the beam path, `recordedBeamLanding`, uses only `len`.
  - The PR-Review reader is retired for pawl. The remaining readers and the tapestry `CommitLog` wiring are untouched.
  - `TestLandingDeps_BeamCommitsMissingLeavesOutAMergeOfTheParent` would fail under the old `CommitLog` wiring, because the parent merge would be listed.
- **F3:** the upstream tip is only visible after the fetch, so `TestHubLandDeps_ParentUnmergedReadsTheUpstreamTip` fails if the fetch or upstream-ancestor check breaks.
- **F4:** `TestBeamLandingRecordSeams_DropRewritesTheRecordAndCommitsTheLandingDirectory` and `TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord` assert the record content, the removed files, the commit subject and exactly one commit, so a wrong pathspec or record drop fails them.
  - The move test relies on the local-path origin making the GitHub state read fail, which the carry path treats as open. This is documented in the test.

Tool calls made: none; my context already held every changed file.
