# beam merge-parent: the merge-in task merges, lands as a true merge

`lym beam merge-parent` files a darn task that brings the parent branch into a beam.
Three things made that task useless; this change fixes each.

- The task's writer was refused `lym fabric merge-in`, `merge-stage` and `merge --continue` by the task-session role guard, so it could not do the one thing the task exists for.
- The task landed through a pull request as a squash, which dropped the parent's commit from the beam's ancestry. The beam's own pull request into the parent then conflicted again.
- The task opened a pull request that was empty whenever the parent's commit was a squash of content the beam already carried.

Closes #64
Closes #65
Closes #66

## What changes

- `fabriccli` role guard: the writer of a merge task may run `merge-in`, `merge-stage`, `merge --continue` and `merge --abort` in its own worktree. The form of `merge` that takes a branch stays Finalize's. Every other task session is refused as before.
- `darnengine` gains `IsMergeTaskSlug`, `MergeTaskSuffix` and `IsMergeTaskWriter`, so the guard and `lym beam merge-parent` share one definition of a merge task. The slug pattern moved out of `beamcli`.
- `boardengine`: `pull_request` accepts a third value, `merge-in` (`PullRequestMergeIn`), beside `on` and `off`.
- `lym beam merge-parent` files the task with `pull_request: merge-in`. Its body and help now say it lands as a true merge with no pull request.
- `landingshed`: `ResolvePullRequest` answers no pull request for a `merge-in` entry, whatever the parent's `beam.yaml` mode and GitHub's rules say, and marks the decision `TrueMerge`. Finalize then never squashes the parent-side merge, so the task branch's merge commit and the parent's commit stay in the beam's history. The GitHub rule lookup is skipped for such a decision.
- `PATTERN-landing.md` and the package docs describe the new value.

## Design decisions

- `merge-in` is a value of the existing `pull_request` field and not a new field. It overrides the parent's mode, including `required` and `merge`, because a pull request would squash away the merged commit. The merge is reviewed again in the beam's own pull request.
- The darn recipe still has its PR-Review row for a merge-in task. The row passes at once, with no pull request and no operator wait, and Describe still runs because its text is the landing commit's message. This deviates from issue #65's "no PR-Review row"; dropping the row would need a separate recipe shape for one instant row.
- The guard keys on the strand name only, as before; it stops a mistaken call and not a session that rewrites the variable.

## To check by hand

- Not run end to end: file a merge task with `lym beam merge-parent` against a beam whose parent has diverged, let it run, and confirm that `git merge-base --is-ancestor <parent commit> <beam>` is true afterwards and that no pull request was opened.
- Check that the merge-in task's run still passes PR-Review and Describe with no pull request record (open finding: the row and Describe still run, see the decision above).
