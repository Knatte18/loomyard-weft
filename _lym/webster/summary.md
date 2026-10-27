# Dobby and adopt hardening

All three batches recorded `done`, with no renames, no pattern departures and no warnings.

## Batch 01: shed-history-machine (head `fd60b7c5`)

Covers cards 1 to 6: the machine stamp on shed history entries, batten telling the shed its machine, landings counted per machine, the one-time emptying of pre-machine landing lists, selfexec before/after-exec callbacks, and the line-atomic, rotating `dobby.log` with raw output moved to `dobby.out`.
Scope guard deviations: `internal/dobbyengine/logfile_test.go`, `internal/dobbyengine/tidy_test.go`.

## Batch 07: dobby-start-and-exits (head `2cc05400`)

Covers cards 7 to 11: lock taken first with a longer wait after re-exec, every dobby exit logged, a beam dobby with a gone worktree ended and its tidy agent stopped, change-only logging with a ten-minute heartbeat, exit tracing before `Run` with raw output on `dobby.out`, and the `adopt` gate key seeded from a tuned finalize in the hub gate file.
Scope guard deviations: `internal/beamcli/dobbyreexec_integration_test.go`, `internal/beamcli/dobbytrace.go`, `internal/beamcli/dobbytrace_integration_test.go`, `internal/configsync/configsync_test.go`, `internal/dobbyengine/loop.go`, `internal/dobbyengine/loop_test.go`, `internal/dobbyengine/loop_worktreegone_test.go`, `internal/dobbyengine/passes.go`, `internal/dobbyengine/passes_test.go`, `internal/dobbyengine/tidy_test.go`, `internal/gateslot/config.go`.

## Batch 12: gate-run-in-process (head `49150a2e`)

Covers cards 12 to 15: in-process gate run with a per-tier observer, the hidden `adopt check` verb, `lym adopt` starting the check and skipping a live one, and the check's progress in `lym beam status`.
Scope guard deviations: `internal/adoptcli/check.go`, `internal/adoptcli/check_test.go`, `internal/adoptcli/checkview.go`.

## Verify gate findings

The first verify attempt failed on two packages, and no recorded card touched either:

- `github.com/Knatte18/loomyard/internal/lymcwd`
- `github.com/Knatte18/loomyard/cmd/lym`

Full log: `.lym/verify/verify-2.log`.
One fixer fork was spawned for them.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `13746825631bb9c7ad35428d26459623ca255ffe` fix: allowlist captured bench prompt links and fit lym adopt Long under the cap
