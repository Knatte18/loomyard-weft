```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F3, F4, F5, F7) and report where one breaks.

**Findings**

1. LOW, F1, `finalize_pr.go` `pullRequestLanding`: a merged scratch pull request now falls through to the resolver, which can drop it and land locally.
   - Case: a beam with no recorded pull request, scratch record `recorded()`, GitHub reports it merged, parent mode `off` or `on`.
   - The override is skipped because the state is not open. `resolveWithParentRule` returns `!Open`, and `landingForDecision` calls `dropRecordedPullRequest`.
   - That function closes nothing for a merged state, removes the record, and returns `prLanding{}`. Finalize then does a local parent-side merge and push of work GitHub already merged.
   - The review's "lands through the already-landed path" holds only when the decision is Open (the `StateMerged` tail of `landingForDecision`). Under `on` or `off` it is a local landing.
   - The origin's parent already holds GitHub's squash commit, so that push may be refused or diverge.
   - The new rows cover only `closed`. The merged branch of the proposal has no row, so I could not confirm it behaves as the review claims.

2. LOW, F7, `retiredmove.go` `recordedPullRequest`: the notice claims "its pull request stays open" for any pull request the landing record holds.
   - Case: the first move found the scratch pull request closed and carried nothing. A repeat finds no scratch record, reads a pull request the landing record held from before, and tells the orch it stays open.
   - The same happens for a beam that never had a scratch record.
   - The proposal never reads the state of the pull request it names.

3. LOW, F3 test `TestHubLandDeps_ParentUnmergedReadsTheUpstreamTip`: it depends on `FABRIC_SKIP_PUSH` being unset.
   - `ParentUnmerged` reads `EnvSyncOptions().SkipPush`, and the test cannot call `t.Setenv` in a parallel test.
   - If the process environment sets it, the upstream branch is skipped and the "want true" row fails with no hint at the cause.
   - The test comment states the dependency but does not guard it.

**Checked, no finding**

- F2 (`BeamCommitsPast` behind `CommitsMissing`):
  - A merged head absent from the line yields the whole line, the same as `MergedLacks`. This only treats the pull request as lacking commits, which is the safe direction.
  - The parent-ancestor check uses the current `refs/heads/<parent>`, the same as the ready side.
  - The task-run `CommitLog` seam is untouched.
- F5: the new halt bullet matches the halt texts in `finalize_pr.go` (the closed-pull-request halt says "the operator reopens it on GitHub, then runs lym loom resume", and `mergeRefusalReason` names `lym beam pr status`).
- F4: the tests exercise the production closures. `HEAD~1` equality pins exactly one commit.

**Tool calls:** none. My context already held every file and diff.
