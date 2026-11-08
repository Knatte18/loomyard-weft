```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it.

Per behaviour:
- **Stencil prose** (parent ruling rule, `Rulings` report section, the fixer-seat no-message sentence, header comments): exempt as static prose. No test needed.
- **`internal/burlerengine/doc.go` prose**: exempt as static prose.
- **`routedDecisionsLine` value** (`internal/burlerengine/prompt.go:238-241`): it is a string constant, so the text is static prose. `fixWork` and its rendering are unchanged. `fix_test.go:282-283` pin that a routed review's work statement contains the constant (`wantWork: routedDecisionsLine`). `fix_test.go:365` pins that an unrouted work statement does not contain it. Both compare against the constant itself, so they hold for the new value and would fail if the routing broke.
- **Code behaviour added**: none. The diff changes no function, branch or error path. `fixWork`, `Fix`, `withoutClass`, `repairReportBeforeGate` and report parsing are untouched.

Findings: none.

One check I did not run: whether any test hard-codes the old constant text. I read `fix_test.go:282`, `283` and `365` from the earlier grep, and all three use the identifier. I did not search the whole tree for the old literal string.

Tool calls: none. My context already held the grep output for `routedDecisionsLine`, the full diff and `fix.go`.
