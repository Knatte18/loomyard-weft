```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. I checked the new landingshed tests and the `LandingMessage` rename; I did not re-run the suite.

- **`CreditOffOpensThePullRequestWithTheDescriptionBodyAlone`:** With `Credit` false, `creditBlock` returns `""` through `Config.CreditBlock`; the `CreditOffIsEmpty` row of `TestConfig_CreditBlock` pins that. `bodyWithCredit("\nChange body\n", "")` then gives `"Change body"`, so `Create` records `"Create task main|Change title|Change body"`. Calls run `FindOpen`, `Create`, `Status 7`. If the empty-credit branch broke and the block were appended anyway, the body would be `wantPRBody` and the `cmp.Diff` would fail.
- **`FailingEnginesRunReadIsAnErrorWithNoPullRequestCall`:** `EnginesRun` fails, so `creditBlock` returns `read the engines the task ran under: %w`. `openPullRequest` wraps that again with `%w`, so `errors.Is(err, readErr)` holds. `creditBlock` runs before `FindOpen`, `Create` and `Edit` and before the record read-back, so `fx.api.calls` stays empty. The push happens earlier, but the test does not assert pushes. If the error were swallowed, `err` would be nil and the test would fail.
- **`LandingMessage`:** only the parameter name and doc changed, so the signature type and behaviour are the same. The caller in `finalize.go` passes the argument positionally, so it is unaffected. The gate run was green (tier0 1837 passed), including the two new subtests.
- **`prcontinue.go` header:** a comment-only change, with nothing to run.

Tool calls made (1):
- `grep -A` of `creditBlock` and `bodyWithCredit` in `internal/landingshed/finalize_pr.go`. My context had the callers but not these bodies, which the two new tests exercise.
