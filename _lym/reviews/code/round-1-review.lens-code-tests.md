```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test row that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test | Result |
|---|---|---|
| Loader accepts `coverage`; scope error names it | `internal/gateslot/config_test.go` (`TestLoadConfig` "an unknown scope" row, and the `templateDefaults` load) | pinned |
| Template keys `darn-write`, `webster-fix`, `code-burler-fix` are `coverage`; `pr-review` is `diff` | `templateDefaults` in `internal/gateslot/config_test.go` | pinned |
| `Targets` split into `Importers`, `BinaryBuilders` and changed packages; `Packages` unchanged | `TestTargetsFor` rows (`fromA`, the `cmd/lym` row, the `cmd/lym` plus `internal/e` row) | pinned, with one gap (F5) |
| `fabricengine.CodeWorktreeChanges` | `TestStatus_MergesUncommittedChangesBothSides_ExcludesWeftArtifacts`, new assertion | pinned |
| `withGateScope` marks a coverage key; a diff key is unmarked; explicit targets stay un-narrowed | `TestWithGateScope` | pinned |
| `coverageTier` returns false for a non-coverage call, a filter flag in the words, `NoCarry`, or a command lacking `{args}` | `TestCoverageTier` | pinned |
| `coverageTier` returns false for an undeclared tier, or a command with no target placeholder | none | gap (F6) |
| `narrowCoverage` runs changed packages and binary builders whole | `TestNarrowCoverage` lines `changed` and `whole: lym binary` | pinned |
| D3 union: selected, unrecorded, out-of-process and guard tests; a tier-1 test excluded from tier 0 | `TestNarrowCoverage` row 1 (`selected 4 of 5`) and row 2 | pinned |
| D3: empty union runs nothing; unwatched guard test; test file constrained to another GOOS | `TestNarrowCoverage` rows 3, 4 and 5 | pinned |
| D3: selector miss runs the package whole | `TestNarrowCoverage` miss row | pinned |
| D3: a test marked unresolved joins the union | none; only `OutOfProcess` is exercised | gap (F6) |
| D3: an unreadable or unparseable test file, or a failed guard scan, gives a `miss: Go file does not parse` run whole | none | gap (F6) |
| D3: a test with no `tiergate.Require` always counts; `tier.Covers` tiers count | none | gap (F6) |
| D6: an over-budget pattern runs the selected packages whole | `TestNarrowCoverage` over-budget row | pinned |
| D6: the second invocation carries an anchored `-run` after the call's words; the first run is dropped when it has no invocation | `TestPlanTierRuns` two new rows | pinned |
| D6: the `-run` filter sits in the run's args, so `emptyTierFailure` still checks the run | none | gap (F3) |
| D6: the green entry carries the `-run` args, so it never serves as a base or a whole pass | none; the summary admits this | gap (F3) |
| D6: `--stop-on-fail`, the failed-test rerun and `tierNumbers` dedupe act on both invocations | none; the `tierNumbers` dedupe change has no row | gap (F3) |
| D7: per-package report lines | `TestNarrowCoverage` (`wantLines`) | pinned |
| D8: `Selection.Recorded` is set from the map | none; `narrowCoverage` tests hand-build `Selection` | gap (F1) |
| D8: `coverageSelections` and `openSelector` (outside a hub gives a no-map miss; unreadable working tree gives a miss; lazy open; per-package cache; `goRootOf`) | none | gap (F2) |
| D1: `computeScope` calls `narrowCoverage` only for narrowed, non-`Whole` targets; under scoper `none` or `dotnet` a coverage key equals `diff` | none; `TestGateTest_Scopes` was only switched to `pr-review` | gap (F4) |

**Findings**

- **F1** — MEDIUM, scope. `internal/testcover/testselect.go:167` sets `Recorded: m.Tests`, and no test in the diff asserts it. Dropping it would make every test count as unrecorded. Every importer would then run all its tests, and no test would fail. Add a `Recorded` assertion to the existing `Select` test, covering a miss (empty) and a usable map.
- **F2** — MEDIUM, scope. `internal/gatecli/coveragescope.go:230-304` (`coverageSelections`, `openSelector`) has no test. Uncovered behaviours: outside a hub the package gets a no-map miss; a `fabricengine` or `go env GOROOT` failure gives a map-unreadable miss for every importer; the selector opens once and selections are cached. This is the D8 "read failure is a miss" criterion. Add a tier0 table test over `coverageSelections` with an empty `coverageMaps`, and an unreadable maps dir if it is reachable without a hub.
- **F3** — MEDIUM, scope. The D6 criteria have no test for the selected run's execution side:
  - its green entry carries the `-run` args and never serves as a base;
  - `emptyTierFailure` still reads the run;
  - `--stop-on-fail` and the rerun act on the second invocation;
  - the `tierNumbers` dedupe in `internal/gatecli/run.go:443`.

  Evidence: `internal/gatecli/tiers_test.go` pins only the plan strings. Add a `greenRecord` row where a selected-run entry is not an ancestor base (`newestWholeAncestor`) and does not satisfy an empty-args `covered`. Add a `TestPlanTierRuns`/`tierNumbers` row where two runs of one tier give one tier number.
- **F4** — MEDIUM, scope. No test drives `computeScope` with a coverage key (`internal/gatecli/scope.go:281`). Unpinned: coverage under scoper `none` or `dotnet` equals `diff`; a `Whole` fallback is not narrowed; explicit targets, `tree_scope` and rerun are not narrowed. `TestWithGateScope` pins only the request marking. Add a `computeScope` row, with a fake selection source if one is injectable. Otherwise extend `TestGateTest_Scopes` with a coverage key under scoper `none` equalling the `pr-review` result.
- **F5** — LOW, scope. `splitUnchanged` (`internal/impactset/graph.go:867`) makes a package reached both through imports and as a binary builder a builder when `cmd/lym` changed. The summary claims this. `TestTargetsFor` has no row where an importing package also builds the binary alongside a `cmd/lym` change. It also has none for a binary-test package that stays an importer when `cmd/lym` is unchanged. Add both rows.
- **F6** — LOW, scope. The `narrowCoverage` and `coverageTier` rows lack these cases:
  - an unresolved-marked test (`marked[name]` is exercised only for `OutOfProcess`);
  - an unreadable or unparseable test file giving the `Go file does not parse` miss;
  - a test with no `tiergate.Require` always counting;
  - `tier.Covers` tiers counting;
  - an undeclared tier;
  - a tier command with no target placeholder.

  Add them as rows of `TestNarrowCoverage` and `TestCoverageTier`.

Test economy and tier rules: no duplicate or per-helper test found. The new tests are tier0, parallel, and spawn nothing.

**Tool calls:** none; the inherited context sufficed.
