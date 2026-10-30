# Land a branch with no commits beyond its parent without a pull request

A run whose task branch holds no commit beyond its parent now lands empty instead of halting at PR-Review.
Before this change PR-Review asked GitHub to open a pull request on such a branch, and GitHub refused with a 422, "No commits between main and <branch>", on every retry.
The halt then told the operator to check the GitHub token, although the call had reached GitHub and been refused on its content.

## What changes

- `ResolvePullRequest` takes a fifth input: whether git finds a task commit the local parent branch lacks.
  When a pull request is wanted but the branch holds none, the answer is no pull request, with a reason naming the empty branch.
  This check comes before any origin fact, so a required parent, an on field or an origin that would otherwise halt does not halt an empty branch.
  A landing that wants no pull request keeps its existing reason.
- `PullRequestDeps` gains `TaskHasCommits`.
  `internal/loomcli` wires it over the task pair through `fabricengine.CommitLog`, from `refs/heads/<parent>` to `HEAD`.
  The check is made by git; GitHub's error text is never read.
  PR-Review requires the seam like its other seams, and Finalize reads it before it calls the resolver.
- On that no, both rows take the no-pull-request path they already had.
  PR-Review closes a recorded open pull request, drops the record and is Done, with no push.
  Finalize does the same with a record and lands locally.
  Its parent-side merge commits nothing, so the board entry is marked done and Worktree-Teardown runs as for any landing.
- The halt text for a failed GitHub call (`githubHaltReason`) now depends on the failure:
  - An unresolvable token names `gh auth login`, as before.
  - A rejected token or an HTTP 401 or 403 response names `GH_TOKEN`, `GITHUB_TOKEN` and `gh auth login`, plus access to the repository.
  - A 422 says that GitHub accepted the token but refused the request's content, gives the empty-branch pull request as an example, and points at the task and parent branches.
  - Any other failure names GitHub's message and `lym loom resume`, without the token.
- `internal/githubclient` gains `ErrTokenRejected` and `StatusCode`.
  The authenticating transport now wraps `ErrTokenRejected` into the error it returns when GitHub answers 401 to an environment-sourced token.
  `StatusCode` reads the status of a `*github.ErrorResponse` in an error chain and returns 0 otherwise, so a rate-limit 403 is never taken for a refused token.
- PR-Review and Finalize check for a token failure, unresolvable or rejected, before the transient classifier, and halt on it.
- `PATTERN-pull-request-mode` names the new input, and the `landingshed` and `githubclient` package docs describe the new behavior.

## Design decisions

- The empty-branch fact goes into the one resolver rather than being checked beside it, so PR-Review and Finalize cannot disagree (`PATTERN-pull-request-mode`).
- The check is "no commit beyond the local parent", not `ParentHoldsTaskWork`.
  That predicate also holds after a squash landing whose task commits still exist, and such a branch can still open a pull request.
  The check uses the local parent because Finalize merges into it, and Finalize's parent sync already refuses a local parent that is ahead of or diverged from its upstream.
- A rejected environment token used to arrive inside a `*url.Error`, which the transient classifier read as transient.
  The row would then have re-stepped forever instead of halting.
  The sentinel lets both rows halt with the token named, without changing `githubclient.IsTransient` for its other consumers.
- There are no parent corrections, recorded or unrecorded.

## Tests

- `TestResolvePullRequest/EmptyTaskBranch` checks the empty branch under an on parent, an on field, a required parent, a required parent with a local origin, and an origin that names no repository: each gives no pull request and no halt.
  It also checks that an off parent keeps its off reason.
- `TestPRReview_Call` checks:
  - an empty branch is Done with no push and no GitHub call;
  - an empty branch with a recorded open pull request closes it and drops the record;
  - the halt texts for a rejected token, a 401, a 403, a 422 and a 404, with the token named only for the first three;
  - an unset `TaskHasCommits` is a wiring error.
- `TestFinalize_PullRequest` checks that an empty branch with no record lands locally with no GitHub call, and that an empty branch with an open record closes it and lands.
  Its 404 row now checks that the text does not name the token.
- `TestPRReview_Wait`'s forbidden-poll row uses a real go-github 403 response.
- `TestStatusCode` covers the status reader, and `TestAuthRT_401Handling` checks that the environment-token 401 wraps `ErrTokenRejected`.

Once this lands, the stuck `label-coverage` run should resume with the fixed binary and land empty.
