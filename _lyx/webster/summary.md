# Origin reconcile and visible push failures

All 16 batches reached `done`; the plan-level verify gate runs after this turn.
Batch 03 failed its first `record-batch` done-check (`TestFabricReplay` did not resolve) and passed after one `recover-batch`.
Every other batch recorded on its first `record-batch`.
The digests carried no warnings.

## Verify gate findings

The gate's first verify failed on six tests:

- `battencli.TestWire_EverySeamFilled`: `Landing.Reconcile` was nil.
- `envkit.TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths`: `NilSeams` reported `Landing.Reconcile`.
- `loomcli.TestLoomResumeReconcilesTheHubConfigBeforeArming`: no hub build stamp after loom resume.
- `loomcli.TestLoomFailedReedUpRefuses/llm_driver`: TempDir cleanup failed, directory not empty.

One fixer fork was spawned for them.

## What was built

- **01** failed-push record store and the installed notice route (cards 1 and 2).
- **03** board replay seam and boardengine's by-entry Bolt replay; recovered once.
- **04** records replay, the ended-set rule and Bolt's pull-first replay (cards 4 and 5).
- **06** `Fabric.Pull` weft replay and reflog judgment of warp.
- **07** Land deploys after a records stop; gitrepo returns a push auth refusal as its own class (cards 7 and 8).
- **09** classified push retry and failure recording.
- **10** failed pushes printed by status and cleanup; the Preflight board push goes detached (cards 10 and 11).
- **12** network-gate entry exemption and fabricengine's local removal with the uncommitted-path snapshot (cards 12 and 13).
- **14** pairteardown local mode.
- **15** origin reconcile types, network calls and branch judgment.
- **16** the moving reconcile.
- **17** ended entries torn down through the seam, stale failed-push records cleared, and pairteardown's reconcile seams (cards 17 and 18).
- **19** `lyx orch start` and `lyx cleanup` reconcile first (cards 19 and 20).
- **21** batten resume reconcile.
- **22** Merge-In and Finalize reconcile.
- **23** hibernate wake reconcile.

## Departures

- Batch 07: `PATTERN-fabric-vocabulary`.
  The card's `weftStop` and `fabricengine.WeftStopError` could not appear in `battencli` or `battenshed`.
  The stop is named `recordsStop` and reached through the neutral exports `RecordsStopMessage` and `NewRecordsStopError`.

No other batch reported a departure.

## Deviations (scope guard)

Files changed outside the batches' own targets, informational:

- 01: `cmd/lyx/destructiveguard_test.go`, `internal/fabricengine/failedpush.go`, `internal/fabricengine/failedpush_test.go`, `internal/orchcli/pushnotifier.go`, `internal/orchcli/pushnotifier_integration_test.go`.
- 03: `internal/boardengine/replay.go`, `internal/boardengine/replay_test.go`, `internal/fabricengine/boardreplay.go`, `internal/testkit/boardkit/boardkit.go`.
- 04: `internal/fabricengine/bolt.go`, `export_test.go`, `replay.go`, `replay_integration_test.go`, `internal/hubreconcile/ensure_integration_test.go`.
- 06: `internal/fabricengine/export_test.go`, `pull.go`, `pull_integration_test.go`, `reflog.go`.
- 07: `internal/battenshed/land_test.go`, `internal/fabricengine/pull.go`, `internal/gitrepo/push.go`.
- 09: `internal/fabriccli/fabric.go`, `internal/fabricengine/boltpullfirst_integration_test.go`, `coalesce.go`, `pushclasses_integration_test.go`, `pushclassified.go`, `recordsgit.go`, `internal/loomcli/commitrecords.go`, `internal/loomcli/wiring_commitstatus_integration_test.go`, `pattern/PATTERN-recipe-setting-lookup.md`.
- 10: `internal/battencli/cleanup.go`, `internal/cleanupcli/segments.go`, `internal/hubforge/hub.go`, `internal/hubreconcile/ensure_integration_test.go`, `internal/hubreconcile/seam_enforcement_test.go`.
- 12: `internal/fabricengine/hubworkspace.go`, `remove.go`, `remove_localmode_integration_test.go`, `snapshotuncommitted.go`, `teardownlock.go`, `internal/pauserecord/hold.go`.
- 14: `internal/pairteardown/teardown.go`, `teardown_test.go`.
- 15: `internal/fabricengine/export_test.go`, `originreconcile.go`, `originreconcile_integration_test.go`, `internal/gitexec/gitexec_test.go`.
- 16: `cmd/lyx/uncontainedwrite_test.go`, `internal/fabricengine/originreconcile.go`, `originreconcile_integration_test.go`, `originreconcilemoves.go`, `originreconcilemoves_integration_test.go`.
- 17: `internal/fabricengine/originreconcile.go`, `originreconcileended.go`, `originreconcileended_integration_test.go`, `originreconcilemoves.go`, `internal/pairteardown/originseams.go`, `originseams_test.go`.
- 19: `internal/cleanupcli/cleanup_integration_test.go`, `reconcile.go`, `internal/orchcli/start.go`, `start_test.go`.
- 21: `internal/battencli/resumereconcile.go`.
- 22: `internal/landingshed/finalize_test.go`, `stuck.go`, `internal/loomcli/landingdeps.go`, `wiring_publish_integration_test.go`, `wiring_seams_test.go`.
- 23: `internal/hibernatecli/reconcile.go`, `internal/hibernateengine/wake.go`, `internal/hibernateengine/world_test.go`.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `050e71761fb49a0923be403b24de35b72b185ef6` fix: treat Landing.Reconcile as nil-legal and keep loom smoke runs' detached board sync from pushing
