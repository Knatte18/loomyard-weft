# gate redundant: a runaway mutant eats all memory and freezes the machine

`redundant-mutant-memory-cap` · Task · bug, gate · starting · Worktree-Create

## Seen (2026-11-16 and 2026-11-17, Neve)

`lym gate redundant`, since its mutation check landed in `e97e2474`, froze Neve twice; both times the power key was the only way out.
The task `redundancy-confirmed-survey` ran it once per package, and both freezes came on `internal/websterengine`.

- On 2026-11-17 four mutant runs started at 07:03:53 (`TestCheckParent`, `TestRecordBatch_ScopeGuardDegradesOnDeltaUnavailable`, `TestRecordBatch_ZeroNewTranscriptsArchivesReport` twice).
  Three held 11.1 GB after 5 s and were killed at the 10 s timeout.
  `TestRecordBatch_AuditOutcomes` then ran alone and reached 13.3 GB in 9 s.
- On 2026-11-16 lym's trace log ends at 15:47:05 with the same four runs open.
- The user slice's 16 GB is `MemoryHigh`, a throttle that kills nothing; `MemoryMax` is 22 GB.
  At `MemoryHigh` the kernel throttles the whole slice, desktop and lym included, so lym stalled before its timeout fired.
  No OOM killer (systemd-oomd, earlyoom) is active.

## Cause (diagnosis read from code and logs, nothing run)

- Mutant runs go through `runMutants` → `runJobs` → `runSideBySide` → `runJob` in `internal/gatecli/redundantmutants.go`, then `Toolchain.RunAlone` and `Toolchain.command` in `internal/testcover/runner.go`.
  Four run at once (tier0 `parallel`), and no rlimit, cgroup or `GOMEMLIMIT` applies anywhere.
- `mutantTimeout` is 10× the unmutated run, with a 10 s floor; a runaway grows about 1–1.5 GB/s, so the timeout cannot bound memory.
- Most plausible mutant: a negated condition on the `for i := 1; i <= a.NamedSpawns; i++` loop in `CheckParent` (`internal/websterengine/audit.go`), which loops for ever when `NamedSpawns` is 0, each pass keeping an `AuditViolation`.
  A second fit is a dropped `elapsed += tick` in `SettleRetry` (same file).
  `RunAlone` logs the test but not the mutant ID, so the logs cannot tell which.

## Fix

- Cap memory on every test-binary run at the choke point, `Toolchain.command`, through a new `Toolchain.MemoryLimit`:
  - Linux: run under `systemd-run --user --scope --quiet --collect -p MemoryMax=<cap> -p MemorySwapMax=0 --`, so the kernel kills the run's tree alone.
    Check that the timeout's process-group kill still reaches the run.
  - Linux without a user manager (a cached probe fails): `RLIMIT_AS` through `prlimit` after start, at parallel 1.
  - Windows: `JOB_OBJECT_LIMIT_JOB_MEMORY` on the job `createKillOnCloseJob` builds in `internal/proc`, which also gives a tree kill on timeout.
  - `GOMEMLIMIT` alone is no fix: it is a soft limit.
- Add `redundancy_test_memory_mb` beside `redundancy_test_timeout_sec` in `internal/gateslot/config.go`, default 2048 (unmutated runs sat near 300 MB).
  Clamp parallel to the memory headroom divided by the cap, reading `memory.high`/`memory.max` up the cgroup ancestry.
- Add `RunResult.MemoryCapped` beside `TimedOut`.
  A capped mutant run counts as a kill, as a timeout does, and is never rerun alone; a capped unmutated run sets `FailsUnmutated`.
  The answer lists capped and timed-out mutants (file, line, operator, test), and each mutant's ID is logged at spawn.
- Lower the mutant timeout floor from 10 s to 2 s.
  In the survey's `internal/loomcli` run, 90 % of its 542 s went to workers waiting out 10 s timeouts; a 2 s floor is estimated to take it from 9 minutes to about 2.5.
- The same `RunAlone` serves `lym gate testmap` and the redundancy report step, so the cap covers them too.
  `cmd/testtiming/redundancy.go` runs with no timeout at all; give it the same bounds.

## Tests

Every allocating fixture stops itself at a few hundred MB, so a broken cap fails the test and never the machine.

- `internal/proc`: a child over the cap is killed, on Linux and on Windows, and the fallback path when the probe fails.
- `TestRunAlone`: a run over the cap returns failed with `MemoryCapped`, not an error.
- `TestGateRedundant`: a fixture with the runaway loop shape finishes, lists the mutant as capped and killed, and is not rerun.
- The parallel clamp, and `TestLoadConfig` for the new key.

## Bounds

The run never runs `lym gate redundant` or `lym gate testmap` on a real package of this repository: only the small fixtures above.
