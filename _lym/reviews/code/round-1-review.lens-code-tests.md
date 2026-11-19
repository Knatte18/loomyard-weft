```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test that fails if it breaks.

**Each behaviour and the test that pins it**

1. **`prReviewSendBackEdge` wires the landing seams before `newShed`, so the shed builds.** Pinned by `TestPRReviewSendBackEdge_BuildsTheShedOfAWiredRun` (`internal/loomcli/prfindings_integration_test.go:20`).
   - Tapestry holds a Finalize row (`internal/loomrecipe/finalizerow.go:17`).
   - Without the `wireLanding` call, `newShed` returns Finalize's nil-OpenFabric error, so the test's `err != nil` check fails on the pre-fix code.
2. **On a wired tapestry run the edge answers true, an empty reason and a nil error.** Pinned by the same test, at line 38.
3. **A recipe with no PR-Review row answers false with a reason, before any fabric is opened.** No test pins this. The diff moves the branch and puts it ahead of `wireLanding`, and the change description says this ordering holds.
   - The row `EdgeOverARecipeWithoutThePRReviewRowIsAnError` (`internal/loomcli/prfindings_test.go:310`) does not cover it. That row injects a fake edge and drives `sendBackEdge`'s find error, which is a different branch with different text.
4. **An error from `wireLanding` reaches the caller.** No test pins this.
5. **`prFindingsCmd` passes `c.prReviewSendBackEdge` as the verb's edge.** No test pins this. The verb's unit tests inject a fake edge (`internal/loomcli/prfindings_test.go:384`), and the new test calls the method directly, not through the command.
6. **The doc.go edits.** Static prose, exempt.

**Findings**

- **LOW, design** — `internal/loomcli/prfindings.go:294` (`prReviewSendBackEdge`, the `!hasRow` branch). No test pins that a recipe with no PR-Review row answers `false, "this run's recipe has no PR-Review row", nil` without wiring the landing seams. `_lym/landing/summary.md` claims this ordering. A tier-1 row could run the method on a wired run of a recipe without the row (sampler or raddle) and assert that exact answer. Evidence: `prfindings.go:294-299`; `prfindings_test.go:310-316` covers only `sendBackEdge`'s find error, through a fake edge.
- **NIT, design** — `internal/loomcli/prfindings.go:298-300`. No test reaches the new error path from `wireLanding`, such as the fabric failing to open. The return is a single plain line, so the risk is low. Evidence: `prfindings.go:298`; no test in `internal/loomcli` makes `wireLanding` fail through this method (grep of `prReviewSendBackEdge` in the package's tests).
- **NIT, design** — `internal/loomcli/prfindings.go:343` (`edge: c.prReviewSendBackEdge`). The seam where the original bug sat is pinned only indirectly. If the command went back to an unwired closure, every test would still pass. Evidence: `prfindings_test.go:384` (fake edge); `prfindings_integration_test.go:37` calls the method directly.

**Tool calls**

- Grepped the package's tests for `no PR-Review row`, `prReviewSendBackEdge` and `edge:`, and `loomrecipe.go` for Finalize. My context did not show which tests touch the edge.
- Grepped `loomrecipe`/`shedrecipe` for Finalize and tapestry, and read `prfindings_test.go:300-320`. This confirmed that tapestry holds Finalize, so the new test fails on the pre-fix code, and showed what the existing no-row test row covers.
