---
round: 1
focus:
  - "internal/beamcli/mergeparent.go `mergeParentBody`: the diff edits the body's second line to read ``Filed by`lym beam merge-parent ...` `` with no space before the backtick. Settle whether the rendered body is malformed and whether a test pins the text."
  - "internal/landingshed `ResolvePullRequest`, `resolveWithParentRule`, `pullRequestLanding`: settle whether `TrueMerge` reaches the parent-side merge on every path a merge-in entry can take, including the beam-record branches (`recordedBeamLanding`) and the already-landed and reopen paths, and whether a merge-in entry can still open or reuse a pull request."
  - "internal/landingshed/finalize.go `landUnderLock`: settle whether `Squash: (Config.Squash || atOnce) && !trueMerge` yields a true merge or fast-forward in the parent pair, and whether `pushParent(..., atOnce || trueMerge)` only changes the protected-branch advice as its doc comment says."
  - "Answer-key item 'no PR-Review row and no GitHub pull request': settle whether a darn task with pull_request merge-in skips the PR-Review row and the Describe step, or whether the diff only changes the landing resolver and leaves those steps running. Search the darn recipe and the run-settings consumers outside the diff for how they read `PullRequest`."
  - "internal/fabriccli/roleguard.go `mergeTaskVerb` and `sandboxedRoleRefusal`: settle whether the exception covers exactly merge-in, merge-stage, merge --continue and merge --abort for a darn writer of a merge-task slug, and that the branch-taking `merge` form stays refused. Check the order of guard checks and `IsMergeTaskWriter`'s strand-name parse (slug shape, role match)."
  - "internal/boardengine/task.go `normalizeTask`: settle whether the accepted `merge-in` value is refused or handled correctly everywhere else `PullRequest` is read or validated (boardcli help, project entries, the doc comment that still says a PullRequest other than empty, on and off is refused)."
  - "Repository rules: check PATTERN.md and pattern/ for the declared changes, the docs each touched package's `doc.go` names, and the `landingshed` seam-enforcement and import-cap edits in `seam_enforcement_test.go` and the board upsert help under the Long cap."
  - "Test economy: check each new test in mergetask_test.go, roleguard_test.go, finalize_pr_test.go, pullrequestmode_test.go, mergeparent_test.go and task_test.go for duplicated coverage, and check that merge-in's landing by true merge (the issue #66 behaviour) has a test that exercises the parent-side merge, not just the resolver."
  - "Change description: check each claim in `_lym/landing/summary.md` against the diff."
---

The answer key is the decision record's Goal alone, three merged issues: #64 (writer may run the merge verbs), #66 (lands as a true merge, never a squash), #65 (no pull request, no PR-Review row).
