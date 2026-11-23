```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
1. **Baseline line now reads "the base of this branch"** (`internal/driftengine/verdict.go:58`). Pinned by the baseline subtest of `TestVerdict` at `internal/driftengine/verdict_test.go:147`, which compares the whole block exactly.
2. **Off scope with a source now reads "this change is not regression-checked"** (`internal/driftengine/verdict.go:100`). Pinned by the `ScopeOffNamesItsSource` row at `internal/driftengine/verdict_test.go:42-45`, also an exact compare.
3. **Off scope with no source now reads "off; this change is not regression-checked."** (`internal/driftengine/verdict.go:102`). No test pins this. `TestVerdict` has only one `ScopeOff` row, and it sets `ScopeSource`; the grep of `verdict_test.go:42-43` shows no other. This is most likely the mutation report's "1 uncovered changed blocks".

**Findings:** none.

- **No finding on item 3:** the branch was already uncovered at the review base, and the diff only rewords it. It adds no new behaviour. The text is a pull request body line that only a person reads. Under the testing rules, wording read only by people gets no wording test, so a new row for it would be a test-economy defect.
- **No loosening:** both changed expectations still compare the whole rendered block with `want`. No assertion was widened or dropped, and each row still checks the case its name gives.

Tool calls:
- grep of `ScopeOff` and `ScopeSource` in `internal/driftengine/verdict_test.go`: my context showed only the diff hunks of that file, not every row, so I needed to confirm that no row covers off scope without a source.
