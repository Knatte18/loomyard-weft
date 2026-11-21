```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it.

Per behaviour, the test that pins it:
1. `RequiresPullRequest` reads a 403 whose message opens `Upgrade to GitHub` as `false, nil`. Pinned by the `plan` row of `TestClient_Calls/RequiresPullRequest` (`internal/pullrequest/client_test.go:278`), whose scripted server answers at `client_test.go:128-129`. Before the change this read returned an error, so the row would fail if the early return were dropped.
2. `RequiresPullRequest` still returns an error for any other 403. Pinned by the `forbidden` row (`client_test.go:279`, server at `130-131`). If the prefix check were dropped, `forbidden` would read as `false, nil` and the row would fail.
3. `RequiredChecks` reads the plan-gated 403 as `nil, nil` and sends only the rules read. Pinned by `client_test.go:310-316`, which checks both the nil result and that the request is the one `rulesPath/plan` read.
4. `RequiredChecks` still returns an error for any other 403 and for a 404. Pinned by the loop at `client_test.go:317-321` over `forbidden` and `unknown`. The `unknown` assertion from the base is kept inside the loop, not loosened.
5. `rulesFeatureUnavailable` requires status 403 together with the prefix (`internal/pullrequest/client.go:33`). The 403 half is pinned only through the `plan` rows. No row sends a non-403 status with an `Upgrade to GitHub` message, so a mutant that dropped the status check would survive. GitHub sends no such answer, so this is no finding.
6. `rulesFeatureUnavailable` returns false when the `*github.ErrorResponse` has a nil `Response` (`client.go:30`). No row covers it, and none can: go-github's `CheckResponse` always sets `Response` on the errors it builds. This is a defensive branch with no behaviour, so no finding under this lens.
7. Finalize lands where it used to halt (`internal/landingshed`). The diff leaves landingshed's code unchanged. The behaviour comes from rows 1-2 composed with the existing `resolveWithParentRule` paths that `internal/landingshed/finalize_pr_test.go:143` and `prreview_test.go:206` already cover through the faked API. No new test is needed.
8. The doc comment changes in `internal/pullrequest/doc.go`, `internal/landingshed/doc.go` and on `RequiresPullRequest`/`RequiredChecks` are static prose, which this lens exempts.

Test economy:
- The new cases are rows and assertions inside the existing `TestClient_Calls` subtests, not new tests, so there is no redundancy.
- No test is tautological: the expected values are literals.
- No assertion targets static prose.
- The redundancy report lists `TestClient_Calls` as unique, and the mutation report shows every mutant killed.

Findings: none.

Tool calls: none.
