```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes on concrete inputs and looked for wrong results, swallowed errors and inconsistent state.

**Functions and branches, the case run, and whether it is correct**

- **`rulesFeatureUnavailable` (`internal/pullrequest/client.go:28`)**
  - `err == nil`: `errors.As` fails, so it returns false. The callers then take their normal `err != nil` path, which is skipped, and continue. Correct.
  - 403 with message "Upgrade to GitHub Pro or make this repository public to enable this feature.": go-github's `CheckResponse` passes this 403 through to its default case. There is no rate-limit header and no abuse documentation URL, so it returns a `*github.ErrorResponse` with `Response` set. `errors.As` finds it through the `url.Error` and `%w` wrapping, the status is 403 and the prefix matches, so it returns true. Correct.
  - 403 with message "Resource not accessible by integration": the prefix does not match, so it returns false. The call fails as before, and landingshed's `githubHaltReason` names the token. Correct.
  - 403 with `X-RateLimit-Remaining: 0`: go-github returns a `*RateLimitError`, not an `*ErrorResponse`, so `errors.As` fails and it returns false. That keeps the earlier behaviour. Correct.
  - 403 with a secondary-rate-limit documentation URL: go-github returns an `*AbuseRateLimitError`, so it returns false. Correct.
  - 401 with an environment token: githubclient's `authRT.RoundTrip` returns an error wrapping `ErrTokenRejected` and no response, so there is no `ErrorResponse` and it returns false. Correct.
  - An `ErrorResponse` with a nil `Response`: the guard returns false. Correct.
  - 404 whose message starts "Upgrade to GitHub": the status is not 403, so it returns false. Correct.
  - Empty message on a 403: `HasPrefix("", prefix)` is false, so it returns false. Correct.
- **`RequiresPullRequest` with branch `plan`:** the first page returns 403 plan-gated, so it returns `false, nil` with no wrap. Correct.
- **`RequiresPullRequest` with branch `guarded`:** page 1 has no pull_request rule, page 2 has one, and the early return is never hit. Correct.
- **`RequiresPullRequest` when page 1 succeeds without a pull_request rule and page 2 returns the plan-gated 403:** it returns `false, nil`, the same as "no rule found". The plan gate is per repository, so this mixed case is unreachable. Correct.
- **`RequiredChecks` with base `plan`:** the first rules page returns 403, so it returns `nil, nil` and reads no check runs or statuses. `ReadMergeState` (`mergestate.go:78`) then sees no required checks. Correct.
- **`RequiredChecks` when page 1 succeeds with contexts and page 2 returns the plan-gated 403:** it returns `nil, nil` and drops the contexts gathered on page 1. That would under-report required checks, but a page-1 success means the repository has the rules feature, so GitHub never sends this answer on page 2. Unreachable, no finding.
- **`RequiredChecks` with base `forbidden` or `unknown`:** it returns a wrapped error, the same as before the change. Correct.
- **`resolveWithParentRule` (`pullrequestmode.go:193`) on a plan-gated repository with `recordFound`:** `required` is false and `err` is nil, so it returns the no decision. Finalize then closes the recorded pull request and lands by direct push. The repository has no rules, so GitHub accepts the push. Correct.
- **`parentRuleRequired` in beamcli (`pr.go:487`) on a plan-gated repository:** it returns `false, nil` and no longer logs the warning. Correct.

**Findings:** none.

**Tool calls:** none; my context already held what I needed.
