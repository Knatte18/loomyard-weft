```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that pins it.

## Per behaviour: the test that pins it

1. **Map run runs a test marked unresolved alone; its entry holds coverage beside the verdict** (`testmappackage.go:129`, `:568`, `:645`): pinned by the `TestCallerUnresolved` row of `TestGateTestMapLinkedPackages` (`testmap_integration_test.go:1748`). The row expects `Blocks: []int{0}` and the `Spawn` field.
2. **A test marked out of process stays unrun with its verdict**: pinned by the `TestCallerSpawns` row of the same test, and by `APackageWhoseTestsAreAllSpawnMarkedIsCheckedBeforeItsWrite`.
3. **`MapTest.HoldsCoverage`, and `buildTable` reaching through it**: pinned by `TestReader`'s `OutcomeAndOutOfProcessEntriesReachNothingAndAnUnresolvedEntryReachesItsBlocks` row (`reader_test.go:157-191`). It expects only `TestUnresolved` in `want`; `TestFailed` and `TestOutOfProcess` both hold block 0.
4. **`mayRunUnseen`**: pinned by the same row for the unresolved, out-of-process, failed and skipped cases.
   - `TestEmpty` (`OutcomeNoCoverage`) pins that a test which covered nothing is left out.
   - Timed out is pinned by `TestReaderIndex`'s `TestG`.
   - The combination unresolved plus `OutcomeNoCoverage` is not pinned (finding 4).
5. **`UnseenTests` sorted by name**: pinned by the same row, whose input order is not name order.
6. **Index carries `Unseen`**: pinned by `TestReaderIndex` (`reader_test.go:493-498`). It compares the answer from a current index against the maps alone and against a literal.
7. **`indexVersion` 2, so an older index reads as absent**: none (finding 3).
8. **`MapFormatVersion` 3**: no reader acts on it. The only assertions compare the marker against the constant itself (`testmap_integration_test.go:1086`, `:1773`), so reverting the bump fails nothing. I name no test gap, since there is no behaviour; whether the bump does anything is the handler's call.
9. **`unseenTestCaveats`, `unseenCause` and `namedTests`**: pinned by `TestUnseenTestCaveats` (`reach_test.go`): one caveat per cause, sorted by cause, outcome before unresolved, and tests past the cap counted. Two gaps: the cap boundary at exactly five, and unresolved plus `OutcomeNoCoverage` (finding 4).
10. **`unmappedTierNames` and the tier caveat text**: only whether a caveat exists is pinned, by `Caveated: true` for `tiered.Two` (`reach_integration_test.go:274`). The text, the tier names, their order, dedup and the "and" join are not pinned (finding 2).
11. **`reachStep.caveats` wiring of `reader.UnseenTests(pkg)`**: none (finding 1).
12. **Doc, `doc.go` and `pattern/` prose**: exempt.

## Findings

1. **MEDIUM**, class `design`, at `internal/gatecli/reach.go:458`: the step-level wiring of unseen tests into a function's caveats has no test.
   - Deleting line 458 passes every test. `TestUnseenTestCaveats` calls the helper directly, and the reach integration fixture's maps hold no unseen test; commit 00eb5125 says so ("the reach step's stand-in maps hold no unseen test").
   - The reach integration test reads caveats only as a bool (`reach_integration_test.go:209`).
   - The reach report lists `reachStep.caveats` as reached by no mapped test.
   - Fix: add a row to the reach integration test whose map holds an unresolved or failed entry, and assert the caveat text on the unreached function.
2. **MEDIUM**, class `design`, at `internal/gatecli/reach.go:455-456` and `:482-495`: the tier caveat's text has no assertion. That text is the board entry's "name the reason".
   - The only check is `Caveated: true` (`reach_integration_test.go:274`), so a caveat that drops or misnames the tier still passes.
   - The reach report lists `unmappedTierNames` as reached by no mapped test.
   - Fix: assert the exact caveat for `tiered.Two`, "tests of tiered in tier2, which no map holds, may run it unseen". Or add a tier0 row with two or more unmapped tiers to pin the order, the dedup and the "and" join.
3. **MEDIUM**, class `design`, at `internal/testreach/index.go:26`: no test pins that an index of another version reads as absent.
   - That rule is what makes a version-1 index fall back to the maps. A version-1 index has no `unseen` key, so read as current it would drop every unseen-test caveat while its stamp matches.
   - `TestReaderIndex` has rows for no index, an index that does not parse, a changed stamp and an added map file, but none for another version (`reader_test.go:395-460`).
   - Fix: add a row that writes a current-stamp index with `"version": 1` and expects the answer from the maps alone, `UnseenTests` included.
4. **LOW**, class `design`, at `internal/gatecli/reach.go:515-523` and `:526-529`: two cases of `TestUnseenTestCaveats` have no row.
   - An unresolved entry with `OutcomeNoCoverage`: `mayRunUnseen` lists it, and `unseenCause` renders "recorded their runs alone as covered nothing, with no coverage".
   - Exactly `maxReachTestsPerFunction` tests: the `&lt;=` boundary of `namedTests`.
   - Fix: add both rows.

Tool calls:
- Searched tests for caveat, tier-caveat and version assertions: my context had no test outside the diff that asserts caveats.
- Read `reach_integration_test.go:150-260`: needed to see how the step's caveats are asserted.
- Read the `TestReaderIndex` row names and the `tiered` fixture lines: needed to check for an index-version row and the tier fixture.
