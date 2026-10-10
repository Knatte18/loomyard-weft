# Driver loop: tied child, compact progress, detached wait, driver stencil

This session resumed a run whose four batches were all recorded `done` before it started, so no batch was begun, forked or recovered here.
`lyx webster status` confirmed every batch terminal with a digest.

- Batch 01 `proc-tied-child`: the child tie, tree kill, pid record and Windows named job in `internal/proc`; `lyx gate test` runs through `proc.StartTied`; loop files under the run's steps directory; `pause_before`/`pause_after` honoured by the engine and cleared by goto; `WriteFailedStop`; `lyx shed pause --before|--after|--clear`.
- Batch 07 `step-progress-compact`: the step envelope's `progress` drops `remaining` and empty keys; PATTERN overview lines shortened with detail in background files; the `bootstrap` arming refusal; `logger.NewTraceID` and `logger.TraceFilesFor`; `runLoop` with children, stop classification, the `loop` object and the transient re-step.
- Batch 12 `loop-detach-wait`: the detached loop, loop lock and pid file, the waiter, envelope currency, the dead-loop stop and `PATTERN-failure-stop-visible`; idle children killed with their tree and reported `interrupted`; `step_idle_timeout_min` in `loom.yaml`; loom fills the loop spec; `lyx loom start` keeps the status strand on llm-driven runs.
- Batch 17 `driver-stencil-rewrite`: the driver stencil around the loop, the guide stencil, the slimmer driver spec and the orch line; a live loop counts as busy and session end kills the loop first; plan-gate findings logged as a count plus a file; the stencil drift notice once per process.

No deviations were reported in this session, since no digest was read.

## Verify gate fixes

The second gate attempt failed on `TestStencilCLI_Scenario/drift_warning_never_blocks_exit_code` in `internal/stencilcli`.
The fixer fork's reply says commit `d3f34221d` fixed it: `stepDriftWarningNeverBlocksExitCode` in `internal/stencilcli/cli_integration_test.go` still asserted the old "board copy has drifted from worktree source" text, and now asserts the shared "drifted from worktree source" fragment of the one-line-per-process warning.
The fork says `lyx gate test --tags integration ./internal/stencilcli ./internal/stencilstore` passes.

The first gate attempt failed on two tests.
The fixer fork's reply says it fixed both in commit `78081ebc1`, and that `lyx gate test` passes on `./cmd/lyx` and `./internal/testkit/envkit`, with and without `--tags integration`.

- `TestSpawnObservability_ProductionSpawnsAreLogged`: `internal/proc/tied.go` cannot import `internal/logger` without an import cycle, so the fork added it to `spawnObservabilityAllowedSpawners` in `cmd/lyx/spawnobservability_test.go` with a reason, and reworded that list's doc comment to not depend on entry order.
- `TestFullEnv_PathFieldsAbsoluteUnderOneRoot`: `envkit.FullEnv` now sets `RunScratchDir` to a `run-scratch` directory under the same temp root, in `internal/testkit/envkit/envkit.go`.


## Background shells waited out

Master's turn end was counted while these background shells were still running, which comes after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; they may still be running in the session.

- `lyx gate test --tags integration ./cmd/lyx ./internal/testkit/envkit 2>&1 | tail -15`
