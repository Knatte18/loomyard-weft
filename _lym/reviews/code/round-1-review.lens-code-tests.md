```yaml
lens: code-tests
outcome: reported
```

**Per behaviour → pinning test**

1. Integer/rune constant rewritten as `(c + 1>>lymMutantStep(id))` — `TestMutantPlan_WriteOverlay` (`internal/testcover/mutant_test.go:360`, expected overlay text holds `names[(0 + 1>>lymMutantStep(14))]`).
2. The mutated constant keeps byte/rune type so the binary compiles, and `lymMutantStep` adds one only while on — `TestMutantBinary_KeepsTheTypeOfAByteContextLiteral` (`internal/testcover/mutantbuild_integration_test.go`): compiles `s[0] == '0'` and `b / 2` with every mutant, wants pass at id 0 and fail at each of 4 mutants; the fixture builder fails the test on a refused build.
3. Compiler error line with no column parsed — `TestBuildMutantBinary` row `NamesTheLinesOfARefusedBuild` (`internal/testcover/runner_test.go:141`, wants `{z.go 62}` from stand-in `/abs/overlay-3/z.go:62: …` at `internal/testcover/testmain_test.go:56`).
4. `UnbuildableMutants` line → file → every-mutant-left blame, none when all dropped — `TestMutantPlan_UnbuildableMutants` (`internal/testcover/mutant_test.go:220-227`): rows for line blame, dropped stays dropped, file fallback, `wholeFiles`, every-mutant-left, none-when-all-dropped, no-line error, two-line mutant.
5. `buildMutants` switches to whole-file blame from round `mutantBuildRounds` on (`round >= mutantBuildRounds`) and no longer fails at the cap — **none**. `TestGateRedundant` (`internal/gatecli/redundant_integration_test.go:340-342`) only reaches a one-round line drop (`Mutants 20, Unbuildable 1`); no test reaches round 5.
6. A line drop in `lym gate redundant` counts the mutant unbuildable and the check finishes — `TestGateRedundant` (`redundant_integration_test.go:341`), its `Timeout` fixture now keeping one unbuildable mutant via an integer in a float context.
7. `ScanTestFuncs` leaves out a test file the default build context does not take — `TestScanTestFuncs` row `TestFunctionsOfTestFiles` via `d_test.go` with `//go:build !<GOOS>` (`internal/testcover/testfuncs_test.go:34-35`). The `build.Default.MatchFile` error path (`testfuncs.go:50`) has no row.
8. Map fixture spawn tests scanned without the tier1 tag — `TestGateTestMapLinkedPackages` (`internal/gatecli/testmap_integration_test.go:1637`), fixture change forced by 7.
9. Embedded-line mutant marks the value after the first `: ` or `=`, else after the indent — `TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine` rows: colon line, equals line, bare key, plain indented line, no mutant. Uncovered branches: (a) a line holding both separators, where the earlier must win; (b) a separator followed only by blanks (`key: ` / `key=`), which must fall back to the indent — the bare-key row `list:` has no `: ` so never enters the separator branch.

**Findings**

- **MEDIUM · design** · `internal/gatecli/redundantmutants.go:410` — No test pins the round-`mutantBuildRounds` switch to whole-file blame, nor that `buildMutants` no longer fails at the cap; reverting `round >= mutantBuildRounds` to the old `round == mutantBuildRounds` failure, or never passing `wholeFiles`, would fail no test. Evidence: only end-to-end assertion is `redundant_integration_test.go:341` (single-round drop); `TestMutantPlan_UnbuildableMutants` covers `UnbuildableMutants` with `wholeFiles` but not the caller's wiring.
- **LOW · design** · `internal/testcover/mutant.go:116-127` (`lymMutantLine`) — The earliest-of-`: `/`=` choice and the blank-value fallback to the indent are uncovered. Evidence: rows at `internal/testcover/mutantbuild_integration_test.go:408-412`; fixture text `"name: lym\nnote = x\nlist:\n\n  plain\n"` at line 314.
- **NIT · design** · `internal/testcover/testfuncs.go:50` — The `MatchFile` error return has no row in `TestScanTestFuncs` (`internal/testcover/testfuncs_test.go:44`); reachable only via an unreadable test file, the same input the existing `ReadFile` error path would cover.

Tool calls:
1. grep of gatecli tests for unbuildable/round assertions — context lacked `redundant_integration_test.go`'s assertions beyond its diff hunks.
