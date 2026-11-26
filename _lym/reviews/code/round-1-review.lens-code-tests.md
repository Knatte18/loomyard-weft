```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it.

**List, per behaviour:**

1. `darn-fragment-make-change.md` step 6: the writer now commits after each coherent step, once that step's gate passes, and never holds the change until the end. This is static stencil prose, so it is exempt and no test pins it. The change adds and removes no marker and no `{{template}}` call, so the existing marker and registry checks in `contracts/stencils` (`markers_test.go`, `registry_test.go`) still cover this file's render contract exactly as they did at the base.
2. `darn-fragment-make-change-gated.md` step 6: the same rule, with the commit following the step's edits. Static stencil prose, exempt, no test. Its markers are unchanged, so the same checks cover it as before.
3. `sampler-template-agent.md` "Working" bullet: "Commit as you go" now forbids holding the work uncommitted until the end and gives the reason. Static stencil prose, exempt, no test. Markers are unchanged, and the `stencil.FillOptional` contract its header comment states is not affected.

No Go code changed. The reach report says "the diff changes no function of the module", and the mutation report shows 0 mutants. The diff therefore adds no behaviour a test could pin beyond static prose, and the rubric exempts prose from the coverage rule. The diff also adds no test, so there is no redundant or tautological test and no test asserting prose to flag.

**Findings:** none.

Tool calls: none. My inherited context already held the diff pieces, the three changed stencils and the reach and mutation reports.
