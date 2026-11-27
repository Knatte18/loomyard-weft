```yaml
lens: code-tests
outcome: reported
```

The code-tests lens is done. One surviving behaviour lost its only tests: the red-first base refusals. Two smaller gaps are LOW.

**Per behaviour, the test that pins it:**
1. `testreach.ChangedTests` now compares declaration text instead of a hash: `TestChangedTests`, rows `NewTestAndChangedTestAreFoundSortedByPackageAndName`, `TestMovedVerbatimToAnotherFileIsNotFound`, `SameNameInTwoFilesIsMatchedPerFile` (new), `DeletedTestIsNotFound`, `ProductionFilesAndFilesOutsideTheModuleArePassedOver` and `UnparsableTestFileIsAnErrorNamingIt`. Covered.
2. `testFileImportPath` with the testdata, dot and underscore exclusions inlined: same test, row `ProductionFilesAndFilesOutsideTheModuleArePassedOver`. Covered.
3. Red-first's `verbDiffBase` refusals (`--base` names no commit; no parent branch and no `--base`): **none**.
4. Red-first's wrapper of the `goModuleOf` refusals ("holds no go.mod", "declares neither tier0 nor tier1"): **none**.
5. A tuned `gates` entry still holding a `reports` list loads, the list ignored: `TestLoadConfig`, row "a tuned key still holding a reports list loads, the list ignored". Covered.
6. A hub file still setting a retired top-level `redundancy_*`, `reach_budget_min` or `mutation_*` key loads: no row in `internal/gateslot` or `internal/configcli`. Only the generic unknown-key path in `internal/configengine` (`layers_test.go`, `set_test.go`) covers it.
7. The process cap counts every marked process, with the carve-out gone: `TestProcessCap`'s remaining rows. Covered.
8. The memory guard kills every test tree, with no foreign filter and no enclosing notes: `TestMemoryGuard` rows; `TestRunTierMemoryGuard`. Covered.
9. `gatefixer` runs the inner gate directly after a fixer: `TestWrapRowGate`, subtest `WriterOnlyVerdictOfTheReRunReturnsToTheWriterWithTheFixersReply` and the entry-fields subtest. Covered.
10. `testcover.Judge` with `JudgeAgainst` folded in: `TestJudge`. Covered.
11. `Toolchain.testCommand` and `goOutput` without `BuildTimeout`/`Ctx`: `TestRunAlone` rows `TimedOut`, `PassWithinTheTimeout` and `ReturnsWithinTheGraceWhileADescendantHoldsTheOutput`; `TestBuildBinaryAndListTests`. Covered.
12. `RunAlone` without `GuardKilled`: `TestRunAlone` rows `Fail`, `MemoryCapped` and `MemoryCappedByTheRuntimesOutOfMemoryLine`. Covered.
13. The rubric fill without the report markers: `TestReadRubric`, `TestLoomRubrics` (with the cut allowlist), `TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`, `TestBouncerEntry_HappyPath`. Covered.
14. The weaver batch gate spec without the reach turn: the tests that call `batchGateSpec` through `weaverengine/export_test.go`. Covered.
15. `testRequest.words()` with `--map` gone: `TestTestRequestWords_MainPackages`, row `TargetsOnly`. Covered.
16. Red-first runs on the new `seedRedFirstModule` fixture: `TestGateRedFirst` and `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged`. Covered.

**Findings:**

- **MEDIUM, design.** `internal/gatecli/redfirst.go:44` (`verbDiffBase`). The red-first verb's two base refusals ("--base %q names no commit; way forward: pass the SHA…" and "the worktree has no parent branch to diff from; way forward: pass --base…") now have no test. Their only rows were `RefusesAPrimeWithNoBase` and `RefusesABaseNamingNoCommit` in the deleted `internal/gatecli/mutate_integration_test.go` (base lines 314–316), and no current `redfirst*_test.go` row asserts either message. That is surviving-behaviour coverage deleted along with tests of removed code. PATTERN-refusal-way-forward also wants a test to reach each refusal's way forward. Fix: add both rows to a `lym gate red-first` refusal test, run from the hub prime with no `--base` and with `--base no-such-commit`, asserting exit 1, the message and the way forward.
- **LOW, design.** `internal/gatecli/gomodule.go:583` (`goModuleOf`), refused at `internal/gatecli/redfirst.go:204-206`. No test drives the "holds no go.mod naming a module inside the worktree" or "the hub declares neither tier0 nor tier1" refusals now. At base the "declares neither tier0 nor tier1" case was reached only through the deleted `internal/gatecli/testmap_integration_test.go` (line 987, the `--map` wrapper of the same helper); no base `redfirst*_test.go` file names either message. Fix: add a red-first row with no tier0/tier1 declared, asserting the refusal and its "declare tier0 or tier1" way forward.
- **LOW, design.** `internal/gateslot/config_test.go`, `TestLoadConfig`. The change description says a hub file that still sets a retired top-level key (`redundancy_budget_min` etc.) still loads, with the unknown-key warning. No gateslot row pins that: grep finds no `redundancy_`, `reach_budget` or `mutation_` in `internal/gateslot/*_test.go` or `internal/configcli/*_test.go`. Only the `reports` case has a row. Fix: add a `TestLoadConfig` row whose hub file sets one of these keys and expects the template config.

Tool calls:
1. Grepped current and base gatecli tests for the moved helpers and the refusal texts: the manifest did not show which tests reach `verbDiffBase` and `goModuleOf`.
2. Listed the `TestChangedTests` rows, the `batchGateSpec` callers and the `TestRunAlone` rows: the manifest held only the diff's hunks of those tests.
3. Read the red-first tests for refusal rows and the base mutate refusal rows: needed to confirm the refusals lost their only tests.
4. Grepped gateslot and configcli tests for the retired top-level keys, and configengine tests for the unknown-key path: needed to see whether loading a retired key is pinned.
