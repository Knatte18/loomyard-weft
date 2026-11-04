```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**
1. A failed capture's reason ends with the run directory and the last three non-empty output lines, or "printed nothing", bounded at `outputTailMaxRunes`.
   - Pinned by `TestRunCapture` outcome rows with `wantEnds`: printed nothing, last-three-non-empty, over-long-line cut. `capture_test.go:~160-195`.
2. The same clause on a main capture that times out.
   - Pinned only by the existing timeout row, `capture_test.go:314`, which checks the `"timed out after "` prefix and the new `wantEnds` suffix check (empty for that row), so it never checks the dir or output clause. **None** for the new text.
3. The `commandReport` branch for a log that cannot be read (`steprun.go:166`).
   - **None.**
4. The accepted capture's reason with the clause before `; log` for exit non-zero and for exit 0 with no file.
   - Pinned by `acceptedRows` in `capture_test.go`. Its timeout row (`:464`) checks only the `"timed out after "` prefix.
5. The compare's reason for exit non-zero.
   - Pinned by `compare_test.go` ("non-zero exit", exact string).
6. The compare's reason for a timeout.
   - `compare_test.go:171` checks only the prefix. **None** for the clause.
7. `RunCapture`'s `OnEnd` hook: called once, with the final record, under the capture lock.
   - Pinned by "OnEnd is called once…".
8. `OnEnd` not called on a refused `Check`.
   - Pinned by the added lines in the refusal subtest.
9. `OnEnd` not called when the command never ran, for example when `failCapture` returns early.
   - **None.** The doc comment claims it, and the early-return paths call `failCapture` directly.
10. `FailedCaptureError`'s text and ways forward.
    - Pinned by `TestFailedCaptureError`.
11. The `runCaptureChild` `OnEnd` hook that writes an error envelope and exits non-zero.
    - **None.** `anchor_integration_test.go:86` covers only the cwd refusal. `capture_integration_test.go` is unchanged.
12. `startMainCapture` failing non-zero when the record already reads failed with the child's pid, and the new `state` field in its result.
    - **None.** No driftcli test was added.
    - The unchanged subtest at `capture_integration_test.go:152-158` asserts `env.OK` for a capture whose marker is absent and which then records failed. If the record already reads failed when the verb returns, this test now fails; if not, it passes by timing.
13. `boundOneLine` replacing `boundChildError` in `spawn.go`.
    - Covered by the existing `spawn_test.go` child-error rows. I did not re-read them.
14. `RebaseDirsOntoRoot`:
    - Rewrite of a dir under the anchor, including `.`, and the case where the anchor wins over the root.
    - Keep of a dir under the root alone.
    - Flag of a dir naming neither.
    - Keep of absolute, unset and undeclared dirs.
    - Same-dir (root-anchored) no-op.
    - Invalid yaml returning `ErrInvalid`.
    - Idempotence.
    - Other keys preserved.
    - All pinned by `TestRebaseDirsOntoRoot` and `_KeepsOtherKeys`.
    - A non-mapping document returning unchanged: **none** (`dirbase.go:352`).
15. `applyReinterpretations`:
    - Rewrite, commit with the marker, and exactly one mail. Pinned by `TestEnsure_DriftDirs…Mailed`.
    - Marker present means no re-apply and no mail. Pinned by the second `Ensure` in that test.
    - Root-anchored hub, no dir set, and absent `drift.yaml` (only the marker is written), with no mail. Pinned by the `…MailsOnlyAFlaggedValue` rows.
    - A flagged value is mailed and kept. Pinned by the same test.
    - A rewrite error (invalid base yaml) failing the walk through `FileError` has no specific row: **none** specific. `TestEnsure_FailedWalk…` is generic, and I did not confirm it uses a drift file.
16. The failed-write restore removing a marker the walk wrote.
    - Pinned by the extended `TestEnsure_FailedWalkNamesTheFile…` loop. It passes only if the failing step comes after `applyReinterpretations`, which I did not verify.
17. `snapshotConfig` covering the markers.
    - Pinned only through item 16.
18. `notifyReinterpretations` failure handling: no notifier installed, notifier error, `!posted`, empty lines.
    - Empty lines are pinned (the no-mail rows).
    - The other three paths have **none**.
19. `orchcli.InstallReconcileNotifier`: the hub-board match, the other-hub `false`, `NotifyPrime` with sender `RECONCILE`, and the `drivableLocation` error.
    - **None.** `notifierwiring_test.go` and `pushnotifier_integration_test.go` were not changed.
    - I grepped them for "Reconcile" and "RECONCILE" and found no match.
20. The `hubLocation` refactor of `InstallPushNotifier`.
    - Covered by the existing `pushnotifier_integration_test.go`.
21. `cmd/lym/main.go` calling `InstallReconcileNotifier` from the root pre-run.
    - **None.** `notifierwiring_test.go` pins the push notifier's wiring; I found nothing for the reconcile one.
22. The registry change: `Module.Reinterpretations` and the drift row.
    - Pinned only through the integration tests.
23. `ReinterpretedMarker` and `ReinterpretedMarkerRel`.
    - Pinned only through the integration tests.

**Findings**

- **MEDIUM, design.** `internal/driftcli/cli.go:349-356` and `:431-435`. The non-zero exit of `lym drift capture` and the non-zero exit and error-envelope write of `capture-child` have no test. That is Part 1's first wanted item. The existing `capture_integration_test.go:152-158` asserts `env.OK` for a failing capture and is now timing-dependent against `startMainCapture`'s new failure branch. Add a row asserting the verb's non-zero exit and its stated cause, and fix or replace the old assertion.
- **MEDIUM, design.** `internal/orchcli/reconcilenotifier.go:19-34` and `cmd/lym/main.go` (the pre-run line). Nothing tests that the hub's orch is mailed, signed `RECONCILE`. The mail is Part 2's second wanted item. Nothing covers the other-hub `false`, the outside-hub no-install, or the pre-run wiring. Existing coverage is `notifierwiring_test.go` for the push notifier only.
- **LOW, design.** `internal/hubreconcile/reinterpret.go:647-665`. The delivery-failure paths have no test: no notifier installed, notifier error, `!posted`. Only the empty-lines silence is pinned.
- **LOW, design.** `internal/driftengine/steprun.go:164-167`, `capture.go:128` and `compare.go:109`. The unreadable-log clause, and the dir and output clause on a timeout (main capture, accepted capture, compare), are unpinned. Only the non-zero-exit and exit-0 wordings are pinned.
- **LOW, design.** `internal/driftengine/capture.go:89,134`. The doc says `OnEnd` is called "once the command ran". No test pins that `OnEnd` is not called on the early `failCapture` returns.
- **LOW, design.** `internal/driftengine/dirbase.go:352`. A non-mapping or empty `drift.yaml` document returning unchanged has no row.
- **LOW, design.** `internal/hubreconcile/reinterpret.go:622-625`. A `Rewrite` error (invalid base yaml) failing the walk through `FileError` has no specific row. `TestEnsure_FailedWalk…` may cover it; I did not read it.

No test duplicates another, and there are no per-helper tests. The `dirbase_test.go` rows and the two `reinterpret_integration_test.go` tests overlap in scenario (root-anchored hub, no dir) but cover different layers, so I report no redundancy.

**Tool calls (3 of 6)**
1. `grep` for the capture-related symbols in `_test.go` files and for "timed out after" tests. The manifest held no test-file search.
2. `grep` in `anchor_integration_test.go` for capture-child and exit assertions, plus an `ls` of the driftcli and orchcli test files. The manifest did not list which driftcli and orchcli tests exist.
3. `Read` of `capture_integration_test.go:120-180`, to see what the existing capture-failure subtest asserts. The manifest did not cover that file.

A fourth call, a grep for "Reconcile" and "RECONCILE" in the orchcli test files, ran inside the `cd internal` command (call 3 in the sequence above counts it with the `driftcli` grep). It found no match.
