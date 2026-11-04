# Land through a squash-merged pull request when the parent is protected

A repository whose `main` refuses direct pushes could not land any task or beam, because Finalize squashed locally and pushed the parent.
A new `beam.yaml` `pull_request` mode, `merge`, lands the run through the remote instead.

## What changes

- With `pull_request: merge` on the receiving branch, Finalize pushes the task or beam branch, finds or opens a pull request into the parent, and squash-merges it on GitHub at once with no review.
  The parent is then pulled in place of the local squash and push.
- The mode is set per parent with `lym config beam --set pull_request=merge` in the parent's worktree, or for every branch of the hub with `--layer default`.
- A `merge` parent always wants a pull request, whatever the landing entry's `pull_request` field says; a field `off` logs the same warning a `required` parent does.
- The parent lock, the gate of the merged tree and the board's done mark are unchanged.
- A parent push the remote refuses as a protected branch is now Stuck with a way forward naming `pull_request=merge`, in place of a bare push failure.
  When the landing already went through a `merge` parent, the way forward leaves out that advice and gives the hand landing only: merge on GitHub, fetch and `git reset --keep` the parent, then `lym loom resume`.
- `dobbyengine` accepts `merge` as a `pull_request` value and its error text lists it.

## Design decisions

- `PullRequestDecision` gains `MergeAtOnce`, set only for a yes under a `merge` parent.
  The review row (`prreview.go`) and the pull-request wait return Done for it with no push, GitHub call or session; Finalize does the opening and merging.
- `openAtOnce` records the pull request with the task head as its gated head and the review marked ended, so a re-entry merges the recorded pull request.
- The merge is always a squash on GitHub, and the parent-side merge squashes too, whatever `landing.yaml` says, so both sides agree.
  A merged recorded pull request under a `merge` parent still squashes parent-side.
- A method refusal from GitHub under a `merge` parent names the repository's squash-merge setting as the way forward, since `Config.Squash` cannot fix it.
- A closed recorded pull request under a `merge` parent is reopened in `mergeOnGitHub`, after the owner checks and the task push, just before the open and the merge.
  Reopening earlier would call GitHub from a machine that may not own the run.
- Protected-branch refusals are recognised by the push error text (`protected branch`, `repository rule violations`), which covers classic protection and repository rules.

## To check by hand

- Land a task and a beam against a real protected `main` with `pull_request: merge`; this was exercised only against the fake GitHub API in tests.
- The reopen of a closed recorded pull request under a `merge` parent now sits in `mergeOnGitHub` (`internal/landingshed/finalize_pr.go`) after the task push; confirm no path still reaches GitHub before the owner checks.
- The protected-branch detection in `protectedBranchMarkers` (`internal/landingshed/finalize.go`) matches GitHub's current rejection wording; confirm it against a real refusal.
