```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**
1. The unit file count replaces the file list, for one file: `TestDoneCheckVerdicts_Rules` (`searched` constant) and `TestDoneChecks_Verdicts` (`donecheck_resolve_test.go:55`).
2. The unit file count for many files: `TestDoneCheckVerdicts_NotFoundDetail/DeclaringSitesCappedNearestFirst`, which expects "2 files of unit internal/a/b".
3. The unit file count for zero files (the `case 0:` branch): not visibly pinned. The "create missing, unit missing" row of `TestDoneCheckVerdicts_Rules` (`donecheck_test.go:94`) probably reaches it, but I did not confirm its declarations or expected detail.
4. The cap of three with "and N more", nearest-first order, and the glyph-order tie-break: `DeclaringSitesCappedNearestFirst`. Four elsewhere-declarations give three named plus "and 1 more". `cmd/x` is named and `pkg/y` is dropped, so the tie-break is pinned.
5. `.`-segment splitting in `sharedUnitSegments` for C# units: none. The test only uses `/`-separated Go units.
6. `signature-mismatch` for a C# Create target with other parameters declared: `NotFoundDetail/SignatureMismatch`.
7. `signature-mismatch` for a Rename new side (the "Rename pair's new side" subject): none. The test only uses `checkID: "create-not-done"`.
8. Negative rows for the mismatch finding: none. Nothing covers a target with no parameter list, a same-name declaration under a different owner or unit, or a Go plan. Each should stay `create-not-done`, and each guards the "only when the same owner and name" requirement.
9. The order of the mismatch check against `declaredInCounterpartPackage`: none. This is only reachable for Go, and Go has no params, so it is effectively moot.
10. Several mismatched declarations passing through the cap in `signatureMismatchFinding`: none.
11. `create-not-done` from record-batch offers both recover-batch and the amendment: `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate`, which asserts the three substrings.
12. `create-not-done` from recover-batch names the amendment alone: `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch`, which also asserts the absence of "to build the target as the card names it".
13. `cardNotDoneFor` setting `createNotDone`: pinned indirectly by 11 and 12.
14. `cardNotDoneFor` setting `signatureMismatch`: none.
15. `failCardNotDone` and `createTargetWayForward` with `signatureMismatch` true (amendment alone from record-batch): none. The summary admits this gap.
16. `createTargetWayForward` with `uncheckable` true (fresh-restart steps): none.
17. Precedence when findings mix (`delete-not-done` referenced, then `glyph-rejected`, then create): none. The existing glyph and delete tests do not mix in a create finding.

**Findings**

- MEDIUM, design. `internal/websterengine/recordbatch.go:222-224`, `createTargetWayForward`. The `signatureMismatch` way forward has no test. `cardNotDoneFor`'s `signature-mismatch` case (`recordbatch.go:907`) is also unpinned. Breaking either, for example dropping the flag, would still pass every test. Issue #61's corrective step lives only here. A tier0 test can build the `cardNotDoneError` with `signatureMismatch: true` and call `failCardNotDone` directly, without a C# fixture. The summary's claim that the recovery test covers this text is wrong: `recovered=true` covers the `recovered` arm, not the `signatureMismatch` arm.
- MEDIUM, design. `recordbatch.go:135-138`. The `uncheckable` branch of `createTargetWayForward` has no test. It is new behaviour. The existing delete and glyph variants are the ones that set the pattern, but no test reaches the create variant.
- LOW, design. `recordbatch.go:208-224`. No test pins the precedence between the delete-referenced, glyph-rejected and create way forwards when findings mix. The `wayForward == ""` guards hold the order.
- LOW, design. `internal/planglyph/donecheck.go:204-208`. The Rename new side branch of `signatureMismatchFinding` has no row. Add a row with `checkID: "rename-not-done-new"` to `TestDoneCheckVerdicts_NotFoundDetail`.
- LOW, design. `donecheck.go:188-199`. The `signatureMismatches` guards have no negative rows. A parameterless target, a different owner, or a different unit declared under the same name should each stay `create-not-done`. These are rows for `TestDoneCheckVerdicts_NotFoundDetail`, not a new test.
- NIT, consistency. `donecheck_test.go:~218-226`. The `NotFoundDetail` table asserts `strings.Contains` on the detail. The testing skill prefers exact equality. The finding's detail is fully deterministic, so the whole string can be compared.

**Test economy.** I found no duplication. `NotFoundDetail` covers behaviours that `TestDoneCheckVerdicts_Rules` and `TestDoneChecks_Verdicts` do not, namely the cap, the ordering and the mismatch. Those two only had their expected strings updated. The new test could have been rows in `TestDoneCheckVerdicts_Rules`, but it is a separate table with its own fixtures, which is acceptable. No new assertion pins static prose.

**Tool calls:** none. The manifest already held every file and test hunk I needed.
