```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I found no finding. The one gap I couldn't close is stencils and specs under `_bolt`, which I did not search. I looked at the following for each change.

**Per-change list**

1. **Failed-command reason now ends with a `; it ran in <dir> …` clause (`capture.go`, `compare.go`, `steprun.go`).**
   - Readers of `rec.Reason` and `result.Reason` are `await.go:88`, `baseline.go:226` and `FailedCaptureError`. They only embed the string, so the longer reason is covered.
   - The tests that assert reason text are `await_test.go:28,108`, `baseline_test.go:409`, `capture_test.go` and `compare_test.go`. The first three use substring or prefix checks and still hold. The last two were updated in the diff. Covered.
   - `boundChildError` was removed. No other reference remains in the repo, and `spawn.go:186` uses `boundOneLine`. Covered.
   - Docs: `driftengine/doc.go`, the `driftcli` help text and `driftcli/doc.go` were updated. Covered.

2. **`lym drift capture` result gains `state` and can fail; the child exits non-zero (`driftcli/cli.go`).**
   - `StartCapture` has two other callers. `loomcli/driftwiring.go:142` discards the pid and ignores state. `baseline.go:198` goes on to Drift-Await, which reads the record and halts with its reason. Neither needs `OnEnd`, and the in-process `RunCapture` fallback at `baseline.go:214` already halts with `record.Reason`. Covered.
   - Tests that spawn `capture-child` are `anchor_integration_test.go:86` and `spawn_test.go`. Both reach the child before the `OnEnd` path, so they are unaffected.
   - No doc, spec or stencil in the repo describes the `{"pid":…}` result shape. Covered.

3. **New `Module.Reinterpretations`, the `_lym/config/reinterpreted-<id>` marker and `applyReinterpretations` (`configreg`, `configengine`, `hubreconcile`).**
   - `snapshotConfig` (`walk.go:433-437`) now includes the markers.
   - `snapshotBoardConfig` (`configcli/hubwide.go:251-259`) snapshots every non-directory file in the config dir, so it already covers a marker. Covered.
   - A marker has no `.yaml` suffix and no module reader lists the config dir.
   - `ensure_integration_test.go` is adapted in two places: `commitPreMigrationBase` and the unpushed-hub-wide-commit changed-file count. Other tests that assert a board commit's file set (`fabriccli/hubmutation_integration_test.go`, `fabricengine/boltpullfirst_integration_test.go`, `configcli/configcli_integration_test.go`) were not diffed. I could not run them, so I cannot confirm they ignore the markers. That is the verify gate's remit.
   - `pattern/PATTERN-config-layers.md:46` says binary-change reconcile goes only through `hubreconcile.Ensure`, which matches the new design.
   - `lym config reconcile` does not apply reinterpretations, which is what the PATTERN line says ("only `hubreconcile`'s walk").

4. **New reconcile notifier (`orchcli/reconcilenotifier.go`, `cmd/lym/main.go`).**
   - `hubLocation` was extracted from `InstallPushNotifier`, which has no other change. `orchcli/doc.go` documents the new installer.
   - `RECONCILE` appears in no registry of program names. `FABRIC` is only a local constant in `orchcli/pushnotifier.go`, and `NotifyPrime` takes the name as an argument. Covered.

5. **PATTERN.md and the declared PATTERN change.**
   - The branch's `PATTERN.md` diff adds `PATTERN-config-reinterpretation` with the text the directive declares, so the declared entry landed.
   - The entry has no `(test)` marker and no background file. Its neighbour `PATTERN-sparse-hub-base` has neither either, so I did not file it.

6. **Commit messages and departures.** The range has no `Pattern-Departure:` trailer, so nothing needs judging.

**Tool calls (6)**
1. Grep `FABRIC` and `fabricProgram` repo-wide: to find any registry or doc of mail program names. My context had none.
2. Grep `boundChildError`, `wrote no file`, `record reads running`, `capture-child` and `InstallPushNotifier`: to find other readers and assertors of the changed strings. My context had none.
3. Grep tests for `BaseSparseMarker`, `PullThenCommitWritten`, `reconcileHubWide` and `ReconcileHubWideAt`, plus `*.md` for "reinterpret": to find other tests and docs touched by the new marker.
4. Grep `ConfigDir(`: to find config-dir listers a marker file could trip. My context had only the two I'd read.
5. Read `configcli/hubwide.go:236-275`: to confirm the board config snapshot covers the marker.
6. Read `pattern/PATTERN-config-layers.md:40-60`: to check its binary-change-reconcile wording.
