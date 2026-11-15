```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests (code-tests lens). For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- `scope: coverage` refused at load, with `lym fabric reconcile` as the way forward: `internal/gateslot/config_test.go:305-308`, row "the retired scope coverage names the migrating way forward". Pinned.
- `MigrateConfig` rewrites `scope: coverage` to `diff` and keeps the other fields and comments: `internal/gateslot/config_test.go:800-804`. Pinned.
- Template keys move to `diff`: `templateDefaults` at `internal/gateslot/config_test.go:80-82`. Pinned.
- Gate key `diff` still runs whole on a fallback, with no coverage narrowing: `internal/gatecli/scopes_integration_test.go:639-645`. The narrowing code is deleted, so nothing is left to pin.
- Beam's retired keys `test_map_upkeep` and `test_map_every: 0` load and change nothing: `internal/dobbyengine/config_test.go:63-64`. Pinned.
- Reconcile prunes `test_map_every` and `test_map_upkeep`: `internal/configsync/configsync_test.go:331-339`. Pinned.
- `lym beam stop` no longer ends an upkeep pass: `internal/orchcli/beamdown_test.go`; the remaining rows assert the exact `wantCalls`. Pinned.
- An anchor move no longer spares an upkeep child: `internal/fabriccli/anchorchild_linux_test.go`; the remaining rows still pin the push and sync spares. Pinned.
- Dobby upkeep step, exit record, `TryAcquireSpare`/`Contended` and `TestFunc.Parallel`: the code is removed, and the test rows for it are removed too. Nothing to pin.
- `--max` has no default cap and caps only when given:
  - `TestGateTestMapCLI/AnUnnamedCallHasNoCapUnlessMaxIsGiven` (`testmap_integration_test.go:1758`)
  - `SecondCallMapsTheRest…` (`:851`).
  - Pinned.
- `--max 0` refusal names the new way forward: `Refusals/MaxBelowOne` (`testmap_integration_test.go:770-771`). Pinned.
- Tests run side by side whatever their `t.Parallel`, and one that fails beside another passes when run alone: `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` (`:1051-1078`). The Meets/Excludes stand-ins fail under serial-only runs and under no re-run alone. Pinned.
- A timed-out test is listed once after its re-run alone: the exact `TimedOut: ["alpha.TestAlphaHangs"]` at `:814`, `:920`, `:940` and `:1029`. Pinned.
- The map is written once per package, and a call ended mid-job writes nothing:
  - `ACallEndedDuringABuildOrATestWritesNothing/WhileATestHangs` (`:1154`, `:1180-1182`); per-round writes would have changed `BuiltAt`.
  - `ALinkedFileChangedUnderAJobDiscardsAllItsRunsAndEndsTheCall` (`:1627-1632`).
  - Pinned.
- Self-check against the job's block table, with the failing entries re-run in a further batch: `ANewTestIsMeasuredAloneAndAKeptEntryTheBuildCannotPlaceIsReMeasured` (`:1427`, `SelfCheckMismatches: 2`). Pinned.
- A probe runs a kept entry when no queued run wrote a profile: `AQueuedTestThatWritesNoProfileStillLetsTheSelfCheckRunAKeptEntry` (`:1444`, `:1458`). Pinned.
- `ProfileByPackage` grouping, nested package included, and `CoveredFuncs` over one group: `internal/testcover/blockkey_test.go:171-183` (`TestCoveredFuncs`). Pinned.
- `ProfiledFuncs` over one group: reached through the self-check subtest at `:1427`. Pinned indirectly.
- An ahead build started for a job that then returns before its build is dropped, and the next package builds fresh: `SecondCallMapsTheRestAndKeepsAFailureAtOneHead` (`:847-885`). `notests` has no mapped test and returns before `takeBuild`; the following `outcomes` map and measures (`:884-885`) would fail on a wrong binary. Pinned indirectly.
- Per-package measures, with tests counted once per job: `:884-885` and `:1615-1616`. Pinned.
- Audit takes its slot through `TryAcquire` only: `TheAuditRunsOnlyWithNothingQueuedAndASlotFreeAndCountsOnlyADifferentResult` (`:1484`). Pinned.
- Named call's build-lock wait, moved to `testmapwait.go`: `ANamedCallWaitsForABuildLockAnotherCallHoldsAndThenRebuildsThePackageWhole` (`:1105`). Pinned.
- A call inside an enclosing gate's slot takes none: `:1080-1103`. Pinned.
- The next package's `go list` and coverage build start while the current package's tests run, and `close` waits for a build still running ahead: **none**.
- A reduce failure (`KeyBlocks` in `newBlockTable`, `CoveredFuncs` or `ProfiledFuncs` in `reduce`) goes through `runBatch` and `measureTests` to `writeFailed`: **none**.

**Findings**

1. **LOW, design (test economy, uncovered behaviour).** `internal/gatecli/testmappackage.go:185-187` (ahead start), `:208-249` (`startBuild`/`takeBuild`) and `internal/gatecli/testmap.go` (`close` waits on `r.ahead.done`).
   - No test pins that the next package's build overlaps the current package's tests. If the ahead start at `:185-187` were deleted, every test would still pass.
   - No test pins that `close` waits for an ahead build still running when the call ends by cancel, HEAD move or error. The cancellation rows at `testmap_integration_test.go:1143-1155` name the single package `alpha`, so no ahead build runs in them.
   - Evidence: the grep of `testmap_integration_test.go` for "ahead" found nothing, and the stand-in records builds only as lines (`:418-425`), with no ordering against test runs.
2. **LOW, design (test economy, uncovered error path).** `internal/gatecli/testmappackage.go:259-277` (`measureTests` turning a `runBatch` error into `writeFailed`), `reduce` and `newBlockTable`.
   - No row makes `KeyBlocks`, `CoveredFuncs` or `ProfiledFuncs` fail inside `reduce`. So nothing pins that such a failure records the package's failure, keeps its entries, and leaves the other workers' profiles dropped.
   - Evidence: the grep for "key the", "name the covered" and "name the fresh" in `testmap_integration_test.go` finds only a comment at `:1425`.

Tool calls:
1. Grepped `testmap_integration_test.go` for `--max`, timed-out, `Packages`, `SelfCheck` and `Audit` assertions: the manifest listed the subtest names but not their assertions.
2. Grepped `testmap_integration_test.go` for reduce-failure text and the build-record env, and read `:1340-1360` and `:805-850`: needed to see whether the build-record and cap rows pin ahead builds or reduce failures.
