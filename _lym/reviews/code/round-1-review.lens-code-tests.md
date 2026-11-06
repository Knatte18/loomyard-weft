```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (or none), as the code-tests lens.

| Behaviour | Pinning test |
|---|---|
| Resume clears `snapshot_lost`, then takes the snapshot | `verify_test.go` "a gone snapshot with no live chair takes a new one…". Its `onArrival` simulates lym's `scope.json`/`judged.json` commits. |
| Guard exempts `scope.json`/`judged.json` while they match their newest `Drift-Records` commit, and exempts the drift logs | `verify_test.go` rows `AScopeRecordCommit`, `AJudgedRecordCommit`, `ACompareLog`; `judgesnapshot_test.go` `DriftLogIsNoChange` |
| A hand edit of either record still halts, then recovers with `--restore` | `AHandEditOf_<name>_StopsAndClearsWithARestore` |
| No `Drift-Records` commit means the record is named | `GuardedRecordNoRecordsCommitWroteIsNamed` |
| A task file or other record the judge edits still halts | The existing `TestJudgeSnapshot` and `TestDriftVerify` change rows |
| Gone snapshot with no live chair: record lost, resume, go on | The "no live chair" subtest |
| Gone snapshot under a live chair stays a Stuck, says so, and the next entry recovers | "under a live chair stops recording it lost…" |
| Snapshot gone while the judge ran stays a Stuck and the next entry recovers | "a snapshot gone while the judge ran…" |
| Changed-checkout halt text names the scratch path and keeps the snapshot directory | `requireStuck(... f.captureDir, "keeps the snapshot directory" ...)` |
| `TakeJudgeSnapshot` clears only the copies and replaces the record atomically | "a new snapshot replaces the record and its copies"; "a snapshot that fails partway leaves the earlier record readable" |
| `openJudgeSession` takes the snapshot before opening the mark | **None** (F1) |
| `volatile_files`/`volatile_keys` load and validate (glob, blank) | `TestLoadConfig` rows (loads, invalid glob, blank file, blank key) |
| Volatile cell dropping: name at any depth, path glob, key, unkeyed line, empty list | `TestVolatile/WithoutCells` |
| Volatile applied in `Check`, with the record's lists or the config's as fallback | Three new `check_test.go` rows |
| Expect listing leaves volatile out under lines only | `TestVolatile/TheListingLeavesThemOutUnderLinesAlone` |
| `baseline.go` records `Volatile` into the scope record | `checkRecordedScope` in `baseline_test.go` (keys only, not files) |
| `verify.go`'s `recordDiffMode` records and keeps `Volatile` | `verify_test.go` recorded-scope assertions |
| `expect.go:125` records `Volatile` when the scope has no diff mode | **None** (F3) |
| `volatile_*` are guard keys, with the hub-only way forward for a task's run settings | **None** (F2) |
| `MigrateConfig` removes the whole `-e`/`--env` argument in every form, in both commands | `TestMigrateConfig_SkipFileArgument`, including the "left alone" rows |
| Reconcile and the new-build walk clear it | `TestEnsure_SkipFileArgumentLeavesTheHubBase` |
| `CheckRetiredPlaceholders`: drift-off, compare only under `diff: command`, the way forward | `TestCheckRetiredPlaceholders` |
| `Preflight.Drift` check, prefix, ordering after the stencils check | `preflight_test.go` new rows |
| `shedrecipe` forwards `PreflightDrift` | "Preflight forwards the drift check" |
| `loomcli` wiring sets `PreflightDrift` for a recipe with a Drift row | `wiring_test.go` subtest |
| Wiring leaves `PreflightDrift` nil for a recipe with no Drift row | **None** (F4) |
| `refuseSkipFilePlaceholder` way-forward text | Fragments asserted in `capture_test.go`, `compare_test.go` and `config_test.go` |

Findings:

- **F1, LOW, design:** Nothing fails if `openJudgeSession` goes back to opening the mark before the snapshot, so the "an open mark always has a snapshot" invariant is unpinned. Evidence: `verify.go:~506-518`. No test in `verify_test.go` makes `TakeJudgeSnapshot` fail (a `Checkouts` error, for example) and then asserts that the mark is not open.
- **F2, LOW, design:** No row checks that `volatile_files` and `volatile_keys` are refused in a task's `run_settings` with the hub's `drift.yaml` as the way forward. Evidence: `config.go` `ConfigSpec`, `CategoryGuard` and `HubOnlyWayForward` entries. The diff adds no test, and the added `config_test.go` rows cover only load and validation.
- **F3, LOW, design:** `DriftExpect.pass` records `scope.Volatile` into a record that has no mode, and no test covers it. Evidence: `expect.go:125` and `:131`. The diff has no `expect_test.go` change, and `verify_test.go` covers only the `recordDiffMode` copy.
- **F4, LOW, design:** The `wiring.go` branch that leaves `preflightDrift` nil when the recipe has no `DriftBaseline` row is untested. Evidence: `wiring_test.go` has only the positive subtest.
- **F5, NIT, design:** The `changedSince` branch for an empty `snapshot.ScratchDir` (the `deletes` text) and the `Seats.Probe` error path in `settleOpenSession` have no row. Evidence: `verify.go:~598-606` and `~565-575`. The fragment test asserts only `f.captureDir`.

Test economy: I found no duplicated coverage. The two snapshot-replacement subtests pin different behaviours.

Tool calls (one):
1. `git diff <base>..HEAD -- '*_test.go'` (filtered to added lines): the manifest held no test bodies.
