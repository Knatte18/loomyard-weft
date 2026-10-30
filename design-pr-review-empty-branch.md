# PR-Review fails with 422 when the task branch has no commits beyond the parent

`pr-review-empty-branch` · Note · bug, landing

## Seen (hub orch, 2026-10-30)

Run `label-coverage` (darn, `pull_request` on) edited no repository file: the labels it added live in the hub `board.yaml`, outside the worktree.
Its branch equalled `main`, so PR-Review failed with GitHub's 422 "No commits between main and label-coverage" on every retry, and the run stopped with a question to the orch.

## Idea

PR-Review (and Finalize) treat a branch with no commits beyond the parent as nothing to land: skip the pull request and finish the landing as an empty one, marking the board entry done.
Open: whether a run that changes no file is itself a fault the darn writer should report, since a task is normally a repository change.
