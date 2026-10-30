## Goal

A run whose task branch has no commits beyond its parent lands empty, with no push and no pull request, instead of halting on GitHub's 422.
A failed GitHub call's way forward names the token only for an authentication failure, and says what a 422 means.

## Scope

- `internal/landingshed`: the pull-request resolver, PR-Review's decision, Finalize's pull-request step and the GitHub halt text.
- `internal/loomcli`: wiring the new git seam over the task pair.
- `internal/githubclient`: a sentinel for a token GitHub rejected, and a reader of a failed call's HTTP status.
- `PATTERN.md`: the `PATTERN-pull-request-mode` entry gains the new input.
- Out of scope: resuming the stuck `label-coverage` run, which the parent does with the fixed binary.

## Decisions

### D1 — The resolver reads whether the task branch has commits

`landingshed.ResolvePullRequest` gains a fifth input: whether the task branch holds a commit its parent branch lacks.
A landing that wants a pull request but whose branch holds none gets no pull request, with a reason naming the empty branch, before any origin fact is judged, so a required parent or an on field does not halt it.
A landing that wants none keeps its existing reason.
Rationale: `PATTERN-pull-request-mode` makes the resolver the one decision, so PR-Review and Finalize cannot disagree; the fact goes in as an input, not as a check beside it.
Bound: only a branch with zero commits beyond its parent skips the pull request; a branch with any commit takes the existing path unchanged.

### D2 — git, never GitHub's error, decides the branch is empty

A new `PullRequestDeps` seam reports whether the task branch holds a commit the local parent branch lacks.
`internal/loomcli` wires it over the task pair through `fabricengine`, listing the commits on `HEAD` not reachable from `refs/heads/<parent>`.
PR-Review and Finalize both read it before calling the resolver, and PR-Review treats it as required wiring like its other seams.
Rationale: the board entry demands a git check; the local parent is the ref the task pair forked from and Finalize merges into.

### D3 — An empty branch takes the existing no-pull-request path in both rows

With the resolver's no, PR-Review closes a recorded open pull request, drops the record and is Done, pushing nothing.
Finalize drops any record and lands locally, where an empty merge already takes the already-landed path: nothing committed on the parent, the board entry marked done, Worktree-Teardown as for any landing.
Rationale: both no paths already exist and are tested; reusing them adds no new landing shape.

### D4 — The halt text matches the failure

`landingshed`'s GitHub halt text names the token (`GH_TOKEN`, `GITHUB_TOKEN`, `gh auth login`) only for an unresolvable token, a token GitHub rejected, or an HTTP 401 or 403 response.
A 422 says GitHub authenticated the call and refused its content as invalid, for example a pull request with no commits between the branches, and names checking the task and parent branches as the way forward.
Any other failure names GitHub's message and `lym loom resume`, without the token.
Rationale: the board entry's design; the 422 that motivated it blamed the token wrongly.

### D5 — A token GitHub rejects is a sentinel, and halts

`internal/githubclient` gains `ErrTokenRejected`, which the authenticating transport wraps into the error it returns for an environment-sourced token GitHub answers with 401, and a reader of a failed call's HTTP status from the go-github error in its chain.
PR-Review and Finalize halt on `ErrTokenRejected` as they do on `ErrTokenUnresolvable`, before the transient check.
Rationale: the transport returns that 401 inside a `*url.Error`, which the transient classifier reads as transient, so without the sentinel the row would re-step a rejected token instead of halting with the token named.
Bound: only the two token sentinels skip the transient check; every other failure is classified as before.

## Constraints

- `PATTERN-pull-request-mode`: the resolver stays the one decision; its PATTERN entry names the new input.
- `PATTERN-fabric-git`: the commit read runs through `fabricengine`, in process.
- `PATTERN-github-auth`: GitHub auth stays in `internal/githubclient`.
- `PATTERN-refusal-way-forward`: each changed halt names a way forward that a test reaches.

## Auto-mode assumptions

- The local parent branch is the right base for the commit check: a parent ahead of or diverged from its upstream is refused by Finalize's sync anyway.
- A record left from an earlier pull request on a branch that is now empty is closed and dropped like any other no.

## Open risks

- Finalize's gate runs over an unchanged tree on an empty landing; the existing already-landed path is trusted to pass it.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - taskhascommits-error-path-untested (class scope, severity MEDIUM, open in rounds 1)
  - taskhascommits-wiring-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: the resolver returns no pull request and no halt for an empty branch under each mode and field that would otherwise want one, including a required parent and an origin that would halt; a non-empty branch keeps every existing answer.
- D2: PR-Review returns an error naming the new seam when it is unset.
- D3: PR-Review with an empty branch under an on parent is Done with no push and no GitHub call, and with a recorded open pull request closes it and drops the record; Finalize with an empty branch under an on parent and no record lands locally with no GitHub call.
- D4: the halt text names the token for a 401 and a 403 response and not for a 404 or a 422; a 422 states what it means.
- D5: a call failing with `ErrTokenRejected` halts naming the token instead of returning a transient error; the transport's environment-token 401 error wraps `ErrTokenRejected`; the status reader returns a wrapped go-github error's status and 0 for an error without one.

## PATTERN changes

- `PATTERN-pull-request-mode` — changes: Deciding whether a landing goes through a pull request: only landingshed's one resolver, from the receiving worktree's `beam.pull_request`, the entry's `pull_request` field, the origin and whether git sees task commits past the parent; no profile or run setting carries it.
