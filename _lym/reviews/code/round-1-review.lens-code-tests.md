```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour**

- Commit target gives the same verdict as the working tree: `TestSelectorSelect` runs every row against both targets (`testselect_test.go:469`). Pinned.
- Commit target whose files cannot be read: none. The fake serves `targetCommit` in every row, so the `CodeFileAt` error path in `readTargetFile` is never exercised. The same goes for `Select`'s error return for a target file that cannot be read.
- Own non-test function body edited selects the tests covering it (`OwnFunctionBodyEdited`), and own Go file removed selects (`OwnGoFileRemoved`): pinned. Own Go file added is no miss (`OwnGoFileAdded`): pinned. A new own `init` misses (`OwnInitAdded`): pinned. An own constant change misses (`OwnConstChanged`): pinned.
- Own non-Go files and testdata: `OwnNonGoFileChanged`, `OwnTestdataFileChanged`, `OwnTestdataGoFileChanged` and `OwnTestFileEmbedsInertFile` pin them. Pinned.
- Test-file comparison:
  - A changed or new `Test` function selects it (`OwnTestFunctionEdited`, `OwnTestFunctionAdded`, `OwnTestFileWithOnlyTestFunctionsAdded`).
  - A removed `Test` function is neither selected nor a miss (`OwnTestFunctionRemoved`).
  - A helper, `TestMain`, an import, a package-level var, a method named like a test, a parse failure or a directive change each miss (`OwnTestHelperEdited`, `OwnTestMainAdded`, `OwnTestImportAdded`, `OwnTestPackageLevelVarAdded`, `OwnTestMethodNamedLikeATestAdded`, `OwnTestFileDoesNotParse`, `OwnTestFileDirectiveRemoved`).
  - Pinned.
  - Not pinned, no test: the package-clause change in `compareTestSources` (`declcompare.go:169`), the `MissCommitUnreadable` return from `compareTestFile` via `readSides` for a recorded test file, and a removed whole test file. These are minor.
- Outcome and spawn entries are no recorded test: `OutcomeEntryIsNoRecordedTest` and the fixture's `TestSpawn` and `TestFailed` entries, checked through the `.Recorded` assertion on every row. A spawn-marked or outcome entry is never selected through coverage: `TestFailed` and `TestSpawn` cover `F`, and `EditInsideFunctionBody` wants only `TestF`. Pinned.
- A new test declared under a spawn-marked name is selected (`SpawnMarkedTestDeclaredInNewFile`): pinned.
- `mapMiss` rejecting a block index outside the table (`TestNamesBlockOutsideTable`): pinned.
- `MissEnvironment`:
  - A differing toolchain version (`ToolchainVersionDiffers`), `go.mod` hash (`GoModChanged`) and `go.sum` added (`GoSumAdded`) are pinned.
  - None for `GOOS`, `GOARCH`, `TiersDigest` or `HubEnvDigest` differing, or for the detail naming the differing fields (`stampDifferences`). The row loop only checks that `Detail` is non-empty.
  - `StampModuleFiles` has no direct test, and none for a `go.mod` or `go.sum` that is absent (empty hash) on either side beyond `GoSumAdded`.
- `gatecli.coverageUnion` running unrecorded tests and no longer checking spawn marks: `TestNarrowCoverage` rows were adjusted. The removed "unresolved" row is now covered by the `TestSelectorSelect` `Recorded` assertion. Pinned.
- `gatecli.stampEnvironment` and `mappedTierConfigs`:
  - No test anywhere under `internal` references either function (grep found none).
  - The acceptance criteria "a changed tier `parallel` value changes no stamp" and "a changed `env.yaml` variable changes it" have no row.
  - Also untested: the tiers digest changing with a tier command or env, and the map build and the gate agreeing on one stamp.
- `gatecli.openSelector` building the stamp through `Toolchain.GoEnvironment`, and its `unreadable` miss when the toolchain cannot be read: none. The only possible coverage is `TestNarrowCoverage` going through a stubbed selector, which never reaches this path.
- `testmap.go` `prepare` now picking `moduleDir` from the lowest mapped tier through `mappedTierConfigs`: none of the diff's tests pin it. Existing testmap tests may cover it, which I did not read.

**Findings**

1. **MEDIUM, design.** `internal/gatecli/testmappackage.go` (`stampEnvironment`, near line 421): no test pins the stamp builder. The D6 criteria "a changed tier `parallel` value changes no stamp" and "a changed `env.yaml` variable changes it" have no row, and nor does the tiers digest following a tier command or env. A table test over `stampEnvironment` is needed. Evidence: no test references it, and the diff adds none in `internal/gatecli`.
2. **MEDIUM, design.** `internal/testcover/testselect_test.go:288-291`: the stamp-miss rows cover only the toolchain version, `go.mod` and `go.sum`. A differing `TiersDigest`, `HubEnvDigest`, `GOOS` or `GOARCH` has no row, and nothing asserts the miss detail names the differing fields (`coveragemap.go` `stampDifferences`). The fix is rows editing `m.Format.Environment` for those fields, plus an assertion on `Detail`.
3. **LOW, design.** `internal/testcover/testselect.go:411-413` and `testselect.go:177-179`: a commit target whose `CodeFileAt` errors, in either `environmentMiss` or `readTargetFile`, has no row. `fakeCommits` always serves `targetCommit`. The fix is a row or case where the target commit is unreadable, asserting `Select` returns an error rather than a miss.
4. **LOW, design.** `internal/gatecli/coveragescope.go` (`openSelector`, the `GoEnvironment` failure path near line 296): the `unreadable` miss for a toolchain that cannot be read is untested.
5. **NIT, design.** `internal/testcover/declcompare.go:169-171`: the `MissTestFile` return for a test file that changes its package clause has no row.

Tool calls:
1. Read `internal/testcover/testselect_test.go`: it was not in my context.
2. grep for `stampEnvironment|mappedTierConfigs|StampModuleFiles|stampDifferences|MissEnvironment` in tests under `internal`: to check for a test of the stamp builder.
