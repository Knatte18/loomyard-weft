```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

The fix changed no function, branch, signature or behaviour: a comment line in `rows_test.go` and three doc comments in `rows.go`.
`go vet ./internal/shellguard` passed before the commits.
The fix added no test.
One point is outside what the context shows: I did not confirm that the `testtiming` tool accepts a marker placed under a doc comment.
The existing markers in `engineterms_test.go` suggest the same layout.

Tool calls made: none. My context already held every changed text.
