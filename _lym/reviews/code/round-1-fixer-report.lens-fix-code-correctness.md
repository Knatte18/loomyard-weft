```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changes (a lens-prose edit, a doc YAML example, a test-row deletion) through the fix-code-correctness lens.

Per-change list (no function or branch was added or changed, so nothing to run step by step):
- `internal/lenses/template.yaml`, the `code-scope` block: the text sits inside a `|` literal block at the same 4-space indentation as its siblings. The backticks, colons, semicolons and em-dash are plain scalar content in a literal block. The gate run for `./internal/lenses` passed 41 of 41 targets, which includes the template-decode test.
- `internal/lenses/config_test.go`: the `"code-scope"` entry was removed from the phrase map. The surrounding literal is still well-formed, since it compiles and the gate passed. `lym loom fmt` reported no changes.
- Test-fails-with-defect check: the fix added no test and the review did not ask for one. Dropping the phrase row loses no behaviour coverage, as the review states. The lens count, the review-code order and the emptied-fan behaviour are still pinned in the same test.
- `docs/dotnet-hub.md`: the YAML fan example now has `code-scope` between `code-tests` and `models-rules`, which matches the template's order.

Findings: no findings.

Calls: none. My context already held every changed file and the gate result, and I made no further tool calls.
