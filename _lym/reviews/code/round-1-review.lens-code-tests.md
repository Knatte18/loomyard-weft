```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: per behaviour, the test that pins it, or none.

**Pinned**

- **Resolver, empty branch.** `TestResolvePullRequest/EmptyTaskBranch` (`pullrequestmode_test.go`) pins it under an on parent, an on field, a required parent, a required parent with a local origin, and a github origin naming no repository. It also pins the off parent keeping its old reason. The existing rows pass `true`, which pins the non-empty path as unchanged.
- **PR-Review empty branch, no push and no GitHub call.** `TestPRReview_Call/EmptyTaskBranchUnderOnIsDoneWithNoPushAndNoGitHubCall` pins it. `wantPushes` is 0 and `wantCalls` is nil, so both are checked.
- **PR-Review empty branch with a recorded open pull request.** `.../EmptyTaskBranchClosesTheRecordedOpenPullRequestAndRemovesTheRecord` pins the close call and the record removal.
- **PR-Review `TaskHasCommits` required.** The `checkWiring` unset-seam map has a `PullRequest.TaskHasCommits` entry.
- **Finalize empty branch with no record.** `TestFinalize_PullRequest/AnEmptyTaskBranchUnderOnWithNoRecordLandsLocallyWithNoGitHubCall` pins the merge, mark and push order and no GitHub call.
- **Finalize empty branch with an open record.** `.../AnEmptyTaskBranchClosesAnOpenRecordedPullRequestAndLandsLocally` pins the close, the record removal and the local landing.
- **`githubHaltReason`.** `TestPRReview_Call` pins the rejected-token-in-`url.Error` case (which also covers `tokenFailure` running before the transient check in PR-Review `githubFailure`), 401, 403, 422 and 404. The token is named for the first three and absent for 422 and 404. The 422 states its meaning.
- **`pollFailure` token path.** `TestPRReview_Wait` has a 403 row that now uses a real `*github.ErrorResponse`.
- **`StatusCode`.** `TestStatusCode` covers a 422, a wrapped 403, a nil response, a `RateLimitError` returning 0, a `url.Error` and nil.
- **Transport 401 wraps `ErrTokenRejected`.** The `TestAuthRT_401Handling` environment-token row has an `errors.Is` assertion.
- **Finalize 404 text.** The 404 row now asserts the token is absent.

**No pinning test**

1. **Finalize `githubFailure` token-sentinel ordering.** Severity MEDIUM, class `scope`. Location: `internal/landingshed/finalize_pr.go:~81`, `githubFailure`. No Finalize row feeds `ErrTokenRejected`, or a rejected-token error wrapped in a `*url.Error`, through the transient check. Evidence: `finalize_pr_test.go` has only the 404 and the existing unresolvable-token rows. Reverting Finalize's `tokenFailure` to the old unresolvable-only check passes every Finalize test. D5 acceptance covers both rows.
2. **Finalize halt text for 401, 403 and 422.** Severity LOW, class `scope`. Location: `finalize_pr_test.go`. Both rows share `githubHaltReason`, and `TestPRReview_Call` pins it, so the split is covered through PR-Review. Only the 404 split is asserted via Finalize. Raise this only if the handler wants Finalize-side pinning.
3. **`TaskHasCommits` error path.** Severity MEDIUM, class `scope`. Location: `prreview.go:84-86` and `finalize_pr.go:121-124`. Neither row has a case where the seam returns an error, which should give an error from PR-Review and `errorStop` from Finalize. Evidence: the fixtures only return `(!taskEmpty, nil)`.
4. **`loomcli.pullRequestDeps` `TaskHasCommits` wiring.** Severity MEDIUM, class `scope`. Location: `internal/loomcli/landingdeps.go:336-344`. The body (`refs/heads/<parent>` to `HEAD`, `len > 0`, the error returned alongside the bool) has no test. Evidence: `wiring_seams_test.go` only lists the seam as intentionally nil, and no test in the diff exercises the closure with a real pair. A swapped `from` and `to` or a wrong ref would pass every test.
5. **`ErrTokenRejected` unwrap through the transient classifier.** Severity LOW, class `scope`. The PR-Review row passes a hand-built `*url.Error` wrapping the sentinel, and `TestAuthRT_401Handling` checks the transport's wrap with `errors.Is`. No test joins the real transport error and the row, so this is pinned only by composition.

Tool calls (1):
- `git diff 9c23208e..HEAD -- '*_test.go'`: my context held only the non-test diff.
