```yaml
lens: code-correctness
outcome: reported
```

Task: correctness lens over the diff; each branch and the case I ran:

- `TaskHasCommits` (`internal/loomcli/landingdeps.go:337`), empty branch (HEAD equals the parent tip): `CommitsNotIn` returns an empty slice, so the result is `(false, nil)`. Correct.
- `TaskHasCommits`, one commit ahead: `CommitDetail` succeeds, so the result is `(true, nil)`. Correct.
- `TaskHasCommits`, a failure in `ResolveSHA("refs/heads/<parent>")`, `CommitsNotIn` or `CommitDetail`: `CommitLog` returns `nil, err`, so `len == 0` and the result is `(false, err)`. Both callers check `err` before reading the bool, in `prreview.go:84` and `finalize_pr.go:121`. The false is never used, so the error is not swallowed. Correct.
- `TaskHasCommits` and a long branch: `CommitLog` makes a `CommitDetail` call for every commit, for a boolean answer. See F1.
- `ResolvePullRequest` empty-branch check (`pullrequestmode.go:73`), each wanted case:
  - An on parent, an on field and a required parent each return `Reason` with no `Open` and no `Halt`, before `checkOrigin`. Correct.
  - A required parent with a field of off still logs the Warn, because the Warn sits at the top of the function before the check. Correct.
  - An off parent with no field keeps its old reason, because the `!wanted` return comes first. Correct.
- PR-Review `decide`, empty branch: the resolver returns no `Open`, so the row takes the existing no path, which closes any recorded open pull request and drops the record. The diff leaves that path unchanged. Correct.
- PR-Review `checkWiring`: an unset `TaskHasCommits` returns a named error. Correct.
- Finalize `pullRequestLanding` with `ParentMode` nil returns before `pr.TaskHasCommits()`. With `ParentMode` set and `TaskHasCommits` nil, it panics. See F2.
- Finalize, empty branch with no record: it takes the no path and lands locally. The merge-adds-nothing path is existing code and I did not re-read it. Correct as far as the diff goes.
- `tokenFailure`, in `githubFailure` of both rows and in `pollFailure`: `errors.Is` reaches `ErrTokenRejected` through `*url.Error` only because the transport formats it with `%w` (`transport.go:142`). Correct. A token rejected with 401 from a cache or gh-CLI source takes the invalidate-and-replay path, not the sentinel, and that path is unchanged.
- `githubHaltReason`, each status:
  - 401, 403 and a rejected token name the token. Correct.
  - 422 names the meaning and the way forward without the token. Correct.
  - 404 and 0 name GitHub's message and `lym loom resume` without the token. Correct.
  - `ErrTokenUnresolvable` is checked first. Correct.
- `StatusCode`: a nil `Response` returns 0, a wrapped `*github.ErrorResponse` returns its status, and a `*github.RateLimitError` or `*github.AbuseRateLimitError` returns 0 because `errors.As` to `*ErrorResponse` misses those types. A rate-limit 403 therefore falls to the generic halt text without the token. This matches the doc comment. Correct.
- `authRT.authenticate`: the 401 error now wraps `ErrTokenRejected`, and the message text changed to "GitHub rejected the token: the token from X (401)". Correct.

Findings:

- F1, LOW, design: `internal/loomcli/landingdeps.go:337-343` answers a yes/no question through `CommitLog`. That makes one `CommitDetail` read per commit on the branch, and builds subject lines that are discarded. `CommitsNotIn` already answers the question alone. A long task branch pays O(commits) reads on every PR-Review entry and every Finalize pass. Evidence: `internal/fabricengine/parenttips.go:152-160`.
- F2, LOW, design: `internal/landingshed/finalize_pr.go:121` calls `pr.TaskHasCommits()` without a nil check. PR-Review's `checkWiring` guards the seam, but Finalize has only the `ParentMode == nil` skip at line 100. Any Finalize wiring that sets `ParentMode` and not `TaskHasCommits` panics. The real wiring (`loomcli`) and `envkit` set both, and the `battencli` and `loomcli` seam-test entries mark the seam as wired or loom-only, so I found no live path that reaches the call with the seam unset. This is a consistency gap with the sibling seams, not a reproduced defect. Evidence: `finalize_pr.go:99-102`, `prreview.go:120`.

Tool calls (2):
1. Read `CommitsNotIn` in `internal/gitrepo/ancestry.go` and the `pullRequestDeps` wiring in `landingdeps.go`, because the manifest lacked `CommitsNotIn`'s empty-slice and error behavior and the `HEAD` and parent-ref wiring.
2. No other calls.
