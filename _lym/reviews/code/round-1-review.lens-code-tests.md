```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds or changes, name the test that pins it.

**List:** The review range `2136b28b..HEAD` is empty. HEAD equals the base `2136b28b`, and `git diff` printed nothing. The diff adds or changes no behaviour, so no behaviour needs a pinning test row. Findings: none.

Two notes outside the lens, for the handler:
- `_lym/landing/summary.md` says the died-driver case passes on base `2136b28b`. I did not run the test, so I cannot confirm that claim.
- The comment above the assertion at `internal/loomcli/smoke_test.go:~728` still calls the text "the read gate's invalid-state refusal". The summary itself lists this as unresolved. That is a comment matter, not a test-coverage one.

Tool calls: none. My context already held the empty diff and the test source.
