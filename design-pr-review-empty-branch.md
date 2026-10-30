# Land a run whose branch has no commits beyond its parent without a pull request

`pr-review-empty-branch` · Task · bug, landing · done

## Seen (hub orch and operator, 2026-10-30)

Run `label-coverage` (darn, `pull_request` on) edited no repository file: the labels it added live in the hub `board.yaml`, outside the worktree, so it has nothing to land.
Its branch equalled `main`, so PR-Review failed with GitHub's 422 "No commits between main and label-coverage" on every retry, and the run stopped with a question to the orch.
The stop report's way forward told the operator to check the GitHub token (`GH_TOKEN`, `GITHUB_TOKEN`, `gh auth login`) although the call reached GitHub and was refused on validation: the text names the token for every failed open.

## Design (operator, 2026-10-30)

A run whose task branch has no commits beyond its parent needs no pull request, and lym must support it: PR-Review skips the push and the pull request, and the landing goes on to finish as an empty landing.
Finalize then lands nothing, marks the board entry done and lets Worktree-Teardown run as for any landing, with no new commit on `main`.
The way-forward text of a failed open names the token only for an authentication failure (401, 403), and says what a 422 means.
The check that the branch has commits is made by git, never by reading GitHub's error.

## Check

The stuck run `label-coverage` resumes and lands empty with the fixed binary.
