```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

- `internal/planglyph/deleteorder_test.go:240` and `internal/planglyph/planfixture_helper_test.go:106` now compare `f.Check` to `planindex.CheckDeleteBeforeReference`. Its value is `"delete-before-reference"` (`internal/planindex/planindex.go:29`), the literal they replaced, so both comparisons behave exactly as before.
- The new `planindex` imports cause no cycle, because non-test `internal/planglyph/index.go` and `deleteorder.go` already import `planindex`.
- `go vet ./internal/planglyph` ran clean in the parent's session. It compiles the test files, so both imports are used and the types match.
- No function signature changed, so no caller is affected.
- The fix added no test, as F3 asked.
- The stencil edits (F1, F2) are prose, not code, and are outside this lens.

Tool calls made: none. My context already held every changed file and the vet result.
