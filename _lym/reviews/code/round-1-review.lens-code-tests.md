```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: per behaviour the diff adds or changes, the test that pins it, or none.

| Behaviour | Pinning test |
|---|---|
| `SyncOptions.Notifier` reaches the failed-push mail (`pushclassified.go` `j.opts.notifier()`) | `TestPushFailureClasses` (`pushclasses_integration_test.go`) passes `fake.options(...)` on every push entry point and asserts the fake's counts. Dropping the option would fail it. |
| `recordPushFailure` mails once per episode, with `notified`, error and deferred handling | `TestFailedPushRecord` rows, through `RecordPushFailureNotifyingForTest` (`failedpush_test.go:45-110`). |
| `recordPushFailure` with a nil notifier mails nothing | The `RecordPushFailure` rows with no notifier installed (`failedpush_test.go:125-187`). |
| `RecordPushFailure` falls back to the installed notifier (`installedNotifier()`) | `TestInstallPushNotifier` (`orchcli/pushnotifier_integration_test.go:113-153`) installs a notifier and calls `RecordPushFailure`. Not verified: I did not read its assertions. |
| `SyncOptions.notifier()` precedence: `Notifier` wins, and a nil `Notifier` falls back to the installed notifier | **None.** Every push test sets `Notifier` explicitly, and no test sets `Notifier` nil with a notifier installed. See F1. |
| `ReconcileOptions.CallHook` on the ls-remote and fetch calls | `TestReconcileOrigin_Judge` rows (`originreconcile_integration_test.go`) pass the hook and assert on `calls`. |
| `ReconcileOptions.VerdictHook` | `TestReconcileOrigin_Judge`, the lock-free-during-verdicts row, asserts `verdicts` and `freeDuringVerdicts`. |
| Nil hooks in `ReconcileOptions` | Every row that passes `ReconcileOptions{}`. |
| `anchorSeams.fail` around the commit and the integration branch move | `TestAnchorChange` rows "failure before/after the move commit", "kill" rows, and the two integration rows (`anchorchange_integration_test.go`). |
| `anchorSeams.weftHead` | `TestAnchorChange`, "A failing read of the commit after the move commit…". |
| `anchorSeams{}` zero value in production, and `moveForkedWeftToAnchor` passing `anchorSeams{}` | All unhooked `TestAnchorChange` rows. The add-anchor path is covered only if a test drives `addanchor.go`; I did not check. |
| `BeginAnchorChange` copies `t.anchor` into `AnchorChange.seams` | Every hooked `TestAnchorChange` row, since each sets the hook on `h.Topology` and would fail if the copy were dropped. |
| `Topology.gcAutoThreshold` is passed to `housekeepStores` (`remove.go:420`) | "an empty answer packs the stores" and "Remove on the default nil probe packs…" (`housekeeping_integration_test.go`) set the threshold to 1 and assert the pack. |
| `NewTopology` sets `gcAutoThreshold` to `defaultGCAutoThreshold` (6700) | **None.** Every test overrides the threshold. See F2. |
| Skip-gc and warning log rows moved to `TestStoreHousekeeping_Warnings` | Same rows, now serial. They still assert keys kept, stores unpacked and the log line. The failing-gc subtest is retained there. |

F1
- Severity: MEDIUM
- Class: design
- Location: `internal/fabricengine/failedpush.go:102-108` (`SyncOptions.notifier`), `internal/fabricengine/pushclassified.go:283`
- Summary: the production fallback is unpinned. A push with `Notifier` nil must mail through the installed notifier, and production passes nil everywhere. A regression that ignores the installed notifier, or that lets nil shadow it, fails no test. Every `TestPushFailureClasses` row sets `Notifier`.
- Evidence: `pushclasses_integration_test.go:34-43` (`newFailureFixture` no longer installs anything) and `pushnotifier_helper_test.go:28-32` (`options` always sets `Notifier`). The only installed-notifier tests are `remoteteardown_test.go` (teardown path) and the `RecordPushFailure` direct path.
- Suggested pin: one row in an existing push test that installs a notifier, pushes with `SyncOptions{}`, and asserts one mail. A second row with both set asserts that `Notifier` wins.

F2
- Severity: LOW
- Class: design
- Location: `internal/fabricengine/topology.go:35-41` (`NewTopology`), `housekeeping.go` (`defaultGCAutoThreshold`)
- Summary: nothing asserts that a `NewTopology` value carries the default threshold. A zero threshold would pass every test, because each one calls `SetGCAutoThresholdForTest(…, 1)`. The zero value would silently disable the foreground gc in production, since `gc.auto=0` disables auto gc.
- Evidence: `housekeeping_integration_test.go` (all rows set the threshold) and `export_test.go:24-27`.
- Suggested pin: an `export_test.go` accessor and a one-line assertion that `NewTopology(cfg)` carries `defaultGCAutoThreshold`.

Tool calls: none; the manifest held everything this lens needed.
