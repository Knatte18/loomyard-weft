```yaml
lens: fix-consistency
outcome: reported
```

No findings on the three fixes' consistency, but one stale twin remains.

**LOW** — `internal/summaryparser/description_test.go:80-92`
- The `TestLandingMessage` table still names its field `trailers`, and its rows `BodyAndTrailers`, `BodyNoTrailers`, `TitleOnlyAndTrailers` and `TitleOnlyNoTrailers`.
- The failure message at line 92 prints `LandingMessage(%q)` with `tt.trailers`.
- F3 renamed the parameter of `LandingMessage` (`internal/summaryparser/summary.go:90`) to `credit`. This test is the one place outside the function that still uses the old name for the same argument.
- Evidence: `grep -rniE "trailer" internal/summaryparser` returns `description_test.go:80`, `:83-86` and `:91-92`.

**Checked and consistent**
- `internal/summaryparser/summary.go:87-91` now says "credit block", matching `internal/landingshed/doc.go:199` ("written as plain lines, never as git trailers").
- The `finalize.go` phrase "gate and credit block" and the new `prcontinue.go` header agree.
- The remaining trailer mentions in `landingshed` are legitimate: the retired config keys, and the description gate's `Co-Authored-By` check.
- The two new `TestPRReview_Call` subtests use `created`, `mergedAtHead` and `githubOrigin`, which are all in scope.
- The new test's expected body `Change body` agrees with `wantPRBody` minus the credit block, so the two stay consistent.

Tool calls: 1. I grepped for "trailer" across `internal/landingshed`, `internal/loomcli` and `internal/summaryparser`, because my context did not show whether the twin test or doc references were stale. A broader first grep over `internal` was too noisy and is not counted.
