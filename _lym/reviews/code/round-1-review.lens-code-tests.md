```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. Each behaviour is listed with the test that pins it, or "none".

**Per behaviour: pinning test**

1. **A refusal whose every blocking finding is delete-before-reference gets `plan_order`, with batches begun or not.**
   - Pinned by `TestBeginBatch_ConflictCauseNamesAPlanOrderConflict`, rows `DeleteOrderAfterABatchIsBegun` and `DeleteOrderWithNoBatchBegun` (`internal/websterengine/beginbatch_test.go`).
   - Each row asserts `Cause`. If `orderOnly` were dropped, the first row would get `tree_moved` and the second `plan_defect`, so both fail.

2. **A blocking set that also holds another finding keeps the old cause.**
   - Pinned by rows `MixedFindingsAfterABatchIsBegun` (expects `tree_moved`) and `MixedFindingsWithNoBatchBegun` (expects `plan_defect`).
   - The second row puts `deleteOrder` last and the first puts it first, so a check on only the first or only the last finding would fail one of them.

3. **An informational delete-before-reference does not count toward `orderOnly`.**
   - Pinned by row `InformationalDeleteOrderBesideABlockingFinding`: `[advisory DBR, blocking glyph-not-found]` must give `tree_moved`.
   - A broken classification that counted informational findings would give `plan_order` and fail.
   - The row does not cover an informational delete-before-reference reaching the switch, because informational findings never enter the blocking branch. That is correct by construction.

4. **`Error()` names the way forward for each cause.**
   - Pinned by `wantWayForward` in every row. `moveDelete` is asserted for `plan_order` and `matchTree` for the other two causes.
   - Both strings include the `rebaseline --card NN` tail, so a lost tail fails.
   - `begin-batch %02d again` is not asserted.
   - It is unchanged, still pinned by the existing begin-batch conflict tests, and not a finding.

5. **`planindex.CheckDeleteBeforeReference` equals `"delete-before-reference"`, so the three `planglyph` emit sites keep emitting the same check name.**
   - Pinned by existing literal comparisons: `internal/planglyph/deleteorder_test.go:240`, `planfixture_helper_test.go:106`, and `webstercli/cli_test.go:268` (`"check":"delete-before-reference"`).
   - A drifted constant value fails these.
   - No new test is needed.

6. **The three-way cause switch.**
   - All three branches (`orderOnly`, no batch begun, batch begun) are covered by rows 1–4.
   - The fourth case, all-DBR with `len(blocking) == 0`, cannot reach the switch because of the `len(blocking) > 0` guard.

7. **`plan_order` flows into the conflict record and the CLI envelope.**
   - `WriteConflictRecord` prints `conflict.Cause` generically, and `webstercli` copies it into the envelope.
   - No test drives `plan_order` through either path. The `webstercli/verbs_test.go` rows use only `plan_defect`, and the comment at `verbs_test.go:411` says the engine's own test covers `tree_moved`.
   - The new engine test does not assert the record or the envelope either.
   - This is a LOW gap at most. The path adds no `plan_order`-specific code, and the generic copy is already pinned for `plan_defect`. I do not raise it.
   - The fix-plan consumers (`planfixer.go`, `ReadConflictRecord`) never read the cause, so no switch there is left uncovered.

8. **Prose changes are exempt from this lens.** These are the stencil, `doc.go`, `loom-plan-checks-spec.md` and the `webstercli` help text. The test adds no assertion on prose.

**Test quality**

- **Duplication:** the new test duplicates no existing coverage. `TestBeginBatch_ReResolvesPlanAtDispatch` and the `webstercli` rows cover `plan_defect` and `tree_moved` through real findings, not through `delete-before-reference` classification.
- **Style:**
  - The `dispatchFindingsIndex` stub embeds `planindex.Index` and overrides only `ValidateDispatch`. That is the line-486 call that produces `resolveFindings`, so it is the right seam.
  - It is a stub for a slow boundary, not a mock asserting calls.
  - The test is parallel and uses `tiergate.Require(t, 0)`.
  - The `tt` loop variable is fine under the Go 1.22 loop semantics `golang-testing` assumes.

**Findings:** none.

**Tool calls**
1. `grep -n "ValidateDispatch\|resolveFindings" internal/websterengine/beginbatch.go`: to confirm that stubbing `Index.ValidateDispatch` is the only source of the blocking findings at the switch. My context showed the diff but not the call site.
