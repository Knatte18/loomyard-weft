# Land a run with no commits beyond its parent without a pull request

A run whose task branch holds no commit its parent branch lacks now lands empty: PR-Review pushes nothing and opens no pull request, and Finalize lands nothing, marks the board entry done and lets Worktree-Teardown run, with no new commit on the parent.
Before, such a run (for example one whose only edits live in the hub `board.yaml`, outside the worktree) halted on GitHub's 422 "No commits between" on every retry.

The halt text of a failed GitHub call also blamed the token for every failure.
It now names the token only for an authentication failure, and says what a 422 means.

## Design decisions

- `landingshed.ResolvePullRequest` takes a fifth input, whether the task branch has a commit its parent lacks.
  A landing that wants a pull request but has no such commit gets no pull request before any origin fact is judged, so a required parent or an `on` field does not halt it.
  A branch with any commit takes the existing path unchanged.
  PR-Review and Finalize both feed it, so they cannot disagree.
- Git decides emptiness, never GitHub's error.
  The new `PullRequestDeps.TaskHasCommits` seam is wired in `internal/loomcli` over the task pair through `fabricengine` (`HeadHoldsCommitsNotIn`), comparing `HEAD` with the local parent branch, the ref Finalize merges into.
  PR-Review requires the seam to be wired.
- An empty branch reuses the existing no-pull-request paths.
  PR-Review closes a recorded open pull request, drops the record and is Done; Finalize drops any record and lands locally through the already-landed path.
- `githubclient` gains `ErrTokenRejected` and `StatusCode`.
  The transport wraps `ErrTokenRejected` into its environment-token 401 error, and `StatusCode` reads the HTTP status from a go-github error in the chain.
  That 401 arrives inside a `*url.Error`, which the transient classifier reads as transient, so PR-Review and Finalize now halt on either token sentinel before the transient check; every other failure is classified as before.
- The halt text names `GH_TOKEN`, `GITHUB_TOKEN` and `gh auth login` for an unresolvable or rejected token and for a 401 or 403.
  A 422 says GitHub accepted the token but refused the request, and points at the task and parent branches.
  Any other failure names GitHub's message and `lym loom resume`.
- `PATTERN-pull-request-mode` names the new input.

## To check by hand

- Resume the stuck `label-coverage` run with the fixed binary; it should land empty with no pull request.
- Finalize's gate runs over an unchanged tree on an empty landing; the existing already-landed path is trusted to pass it, and no test runs that combination end to end.
