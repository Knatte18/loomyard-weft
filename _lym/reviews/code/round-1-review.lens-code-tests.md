```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour: the test that pins it, or none**

- **Finding `//lym:only` markers in a file** (keys sorted, repeats dropped, tier read, stacking with other directive lines, misplaced, malformed, parse error): pinned by `TestFindOnlyTests` rows, `internal/impactset/only_test.go:308-344`.
- **`ModuleOnlyTests` over a real module, and the `modulePackageDirs` split**: pinned by `TestModuleGuardTests_OverAModule` subtest "the only-marked tests…", `internal/impactset/guardscan_integration_test.go:114`. The existing guard subtest pins `ModuleGuardTests` after the refactor.
- **`InsideGuards` split at the set, with unwatched inside guards dropped**: pinned by `TestTargetsFor` rows, `internal/impactset/targets_test.go:54,182`.
- **Guard step of a computed scope taking an inside guard whose tier is not listed**: pinned by `TestGateTest_Scopes` subtest, `internal/gatecli/scopes_integration_test.go:562-573`, for both the default call and `--tier 1`.
  - Not pinned: a listed tier that only *covers* the guard's tier (`skippedInside`'s `Covers` branch, `scope.go:76-78`), and the `--base` form.
- **`withOnlyTests`: held list, unknown key refused, a call without `--gate` holds nothing**: pinned by the same subtest, `scopes_integration_test.go:575-592`.
- **`withOnlyTests` scanning the `-C` directory of explicit targets**: none.
- **`withOnlyTests` when `go list` fails** (warning, nothing held back; a marked key refused instead): none.
- **Held tests joining the listed runs' `-skip`, with recorded args excluding them**: pinned by the `TestPlanTierRuns` rows at `tiers_test.go:466-488`, and the filter-flag row at `:489`.
- **`withoutHeld`: guard step leaves out a guard test whose marker does not name the key**: none.
- **Marked scope runs only its marked tests**: pinned by the `--gate upkeep` row, `scopes_integration_test.go:587`.
- **Marked scope: a marked test whose tier the key does not list is named on stderr and not run**: none.
- **Marked scope refused under a scoper other than go**: none.
- **Explicit targets replacing a marked key** (`isMarkedCall` false with targets): none.
- **`covered()` and `newestWholeAncestor()` honouring `held`**: pinned by `TestGreenRecord` rows `green_test.go:233-235` and `TestGreenRecord_NewestWholeAncestor` rows `:280-281`.
- **`greenRecord.with` treating entries with different `Held` as different keys**: none.
- **`recordPass` and `recordInertCarried` writing `Held` into the entry**: none. `green_integration_test.go:51` counts entries only.
- **gateslot: scope `marked` accepted, `upkeep` in the template, the refusal names `marked`**: pinned by `templateDefaults` at `gateslot/config_test.go:83` and the unknown-scope row at `:329`.
- **`TidyPrompt` with and without an upkeep call**: pinned by `TestTidyPrompt`, `internal/dobbyengine/tidy_test.go:398-411`.
- **`daemon.upkeepCall`** (nil func gives an empty call; a render error is logged and gives an empty call) **and `spawnTidy` passing the call**: none.
- **beamcli: `UpkeepCall` set only on the prime's dobby (`geom.Slug == ""`), rendering `AgentGateCall(upkeep)`**: none.
- **Removing `create-caller-later`**: its tests went with the check (`latercaller_test.go`, `testsupportdirs_linux_test.go`). No behaviour is left to pin.

**Findings**

1. **MEDIUM**, class `design`, `internal/gatecli/only.go:408` (`withoutHeld`). No test covers the guard step leaving out a held guard test. This is the behaviour every `cmd/lym` guard with `//lym:guard` and `//lym:only parent-align finalize` relies on to leave task gates. The only fixture with a marker (`scopes_integration_test.go:453`) carries no `//lym:guard`, and its `finalize` row runs scope `all`, which plans no guard step. Deleting `run.go:502` (`sc.guards = withoutHeld(...)`) would pass every test.

2. **MEDIUM**, class `design`, `internal/gatecli/green.go:200,209` (`recordInertCarried`, `recordPass`). No test asserts that a recorded green entry carries `Held`. The `covered` and `newestWholeAncestor` rows build `Held` by hand (`green_test.go:93,308`), and `green_integration_test.go:51` checks only the entry count. If the writer dropped `Held`, a webster pass that skipped the landing checks would cover Finalize, and no test would fail.

3. **MEDIUM**, class `design`, `internal/gatecli/only.go:386-390` (`markedScope`). No test covers a marked test whose tier the key does not list. That is the template's case for this hub (`upkeep: {tiers: [0]}`, upkeep tests in tier 1): the tests are named on stderr and skipped. No row checks that line, or what such a run reports.

4. **MEDIUM**, class `design`, `internal/beamcli/dobbydeps.go:409-417` and `internal/dobbyengine/tidy.go:681-691`. No test covers wiring `UpkeepCall` on the prime alone, nor `daemon.upkeepCall`'s nil and render-error fallback to the no-upkeep step. `grep UpkeepCall` over `*_test.go` finds only `tidy_test.go`.

5. **LOW**, class `design`, `internal/gatecli/only.go:342-347` (`withOnlyTests`). No test covers a failed `go list`: the warning path that holds nothing back, and the refusal of a marked call.

6. **LOW**, class `design`, `internal/gatecli/only.go:377-379` (`markedScope`). No test covers the refusal under a scoper other than go.

7. **LOW**, class `design`, `internal/gatecli/green.go:124` (`greenRecord.with`). No test covers `sameKey` now comparing `Held`, so that entries with different held sets are kept apart instead of replacing each other.

8. **LOW**, class `design`, `internal/gatecli/scope.go:73-78` (`skippedInside`). No test covers a listed tier that covers the guard's tier, which must keep the guard out of the guard step. The integration rows exercise only a directly listed tier.

9. **LOW**, class `design`, `internal/gatecli/only.go:332` and `scope.go:67-69`. No test covers `withOnlyTests` scanning the `-C` directory of explicit targets, nor explicit targets replacing a marked key.

Tool calls:
- Grepped `*_test.go` for the stderr lines of the marked scope and scan failure, `Held`, `withoutHeld`, `UpkeepCall` and `OnlyMalformed`. My context held the diff's tests, not the existing test files.
- Grepped the gatecli tests for green `with`, `recordPass` and inert-carry usage, the `//lym:guard` lines in the scopes fixture, and its upkeep key. I needed to know whether the existing tests pin recorded `Held` and a held guard.
