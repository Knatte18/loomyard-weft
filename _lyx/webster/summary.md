# Run process ownership: one owner record for every run process

All 14 batches reached `done`, ending at head `e6b6520bdcfa114889877f2398860d1c40062811`.
Batch 07 failed its first `record-batch` on two test targets that did not resolve and finished through one `recover-batch`.
No digest listed `departures`, and no `warnings` came back.
The scope guard listed `deviations` for every batch; each lists files the batch touched beyond its card targets, informational only.

## What was built

- **01 proc-stat-reads** (cards 1 and 2): `internal/proc` reads process stats and signals a pid or group, and the new leaf package `internal/runproc` holds the owner record and `End`.
  Deviations: `internal/lyxcwd/enforcement_test.go`, `internal/proc/proc_linux.go`, and the `internal/runproc` files.
- **03 register-detached-spawns**: detached pushes and board syncs register under the owner names.
  Deviation: `internal/boardengine/spawn.go`.
- **04 register-run-roles** (cards 4 and 5): the driver, the step loops and a seeded batten register under their slug, and the network gate leaves a waiting marker and can exempt every entry for one process.
  Deviations: `internal/battencli/arm_seed_test.go`, `internal/pauserecord/hold.go`.
- **06 deferrable-gate**: the gate of a detached push or board-sync child defers instead of waiting.
  Deviations: `internal/pauserecord/hold.go`, `internal/tendercli/cli_test.go`.
- **07 detached-children-defer**: detached pushes and board syncs exit under a pause, flags and commits kept.
  Recovered once.
  Deviations: `internal/fabricengine/export_test.go`, `internal/fabricengine/spawn.go`.
- **08 lift-spawns-owed**: every lift spawns the pushes and the board sync a pause deferred.
  Deviations: `internal/tenderengine/pause_test.go`, `internal/tenderengine/resume.go`.
- **09 teardown-ends-owned**: pair teardown ends the pair's owned processes before its loop and session.
  Deviations: `internal/pairteardown/teardown.go`, `internal/pairteardown/teardown_test.go`.
- **10 teardown-skip-readings**: teardown skips the driver checks of a run with nothing left to do and waits for the run lock first.
  Deviations: `internal/pairteardown/originseams.go`, `teardown.go`, `teardown_test.go`.
- **11 fabric-remove-reconcile** (cards 11 and 12): `lyx fabric remove` reconciles first and removes an ended or foreign run locally, and `lyx reed down` ends a pair's owned processes first.
  Deviations: `internal/fabriccli/remove.go`, `remove_test.go`, `internal/pairteardown/claim.go`, `skip.go`, `internal/reedcli/up.go`.
- **13 owned-elsewhere-end**: an owned-elsewhere pause ends the run's processes on the machine it left.
  Deviation: `internal/hibernateengine/world_test.go`.
- **14 tender-status-processes**: `lyx tender status` lists owned processes and orphans.
  Deviations: `internal/tenderengine/pause_test.go`, `processes.go`.
- **15 tender-reap**: `lyx tender reap` ends the orphans of one fresh reading.
  Deviations: `internal/tenderengine/processes.go`, `reap.go`, `reap_test.go`.
- **16 hibernate-confirm-background**: hibernate confirmation settles shell-only background work and waits out forks.
  Deviation: `internal/hibernateengine/watch.go`.
- **17 shuttle-shell-only-waiting**: shell-only background work never holds a row its outputs settle.
  Deviations: `internal/shuttleengine/claudeengine/corpus_test.go`, `internal/shuttleengine/completionsignal_enforcement_test.go`.
