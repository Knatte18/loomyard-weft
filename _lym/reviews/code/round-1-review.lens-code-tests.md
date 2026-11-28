```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **Gated Step 4 fragment bars subagents from measurement calls** (`contracts/stencils/darn/darn-fragment-make-change-gated.md:46`). This is static stencil prose, so the lens exempts it. A row does pin it: the gated row of `TestDarnSpec` ("a cold fixer on the verify gate has the writer and its subagents run no gate") asserts `measureNoSubagent` (`internal/darnengine/spec_test.go:154` in the diff).
2. **No-parent way forward names `lym loom decision add` then `lym loom resume`, or a board note once Code-Review has run** (`darn-fragment-make-change.md:38-40`, `darn-fragment-make-change-gated.md:54-56`). This is static stencil prose, so the lens exempts it. A row does pin it: both `TestDarnSpec` rows assert `measureNoParentWayForward` (`spec_test.go:126` and `:154` in the diff). Only the first clause of the sentence is pinned. The closed-scope clause and the `then lym loom resume` tail have no assertion, which is fine because prose is exempt.
3. **Code behaviour.** The diff changes no Go production code. `DarnTable` still picks the fragment the way it did, and the existing `ungatedTestStep`, `ungatedSubagentGate` and `gatedNoGateStep` assertions in the same rows already pin which fragment is chosen (`spec_test.go:78-83`). The stop-marker handling in `internal/loomshed/darnwrite.go:189-211` and the scope-closed refusal in `internal/loomcli/decision.go:378-409` are reached by the new wording but not changed. Their existing tests cover them, and this diff adds nothing that needs a new row.

**Findings: none.** Every behaviour the diff adds is stencil prose, which the lens exempts. No code behaviour lacks a pinning row.

One thing outside my lens: the two new `TestDarnSpec` assertions only check static stencil prose, and that prose now appears in both fragments, so neither assertion tells the gated and ungated rows apart. That falls under the rubric's test-economy bullet on prose assertions, which is the handler's or another lens's call.

Tool calls: none.
