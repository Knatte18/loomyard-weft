```yaml
lens: code-correctness
outcome: reported
```

Lens task: check that every function and branch the diff adds or changes behaves correctly, running each with one concrete input.

**Functions and branches, the case I ran, and the result**

- `proc.runMemoryCapped`, scope path, a child that passes `MemoryMax`: the kernel SIGKILLs the exec'd test binary and `cancelled` is false, so it returns `killed=true` with an `ExitError`. Correct.
- `proc.runMemoryCapped`, scope path, a timeout cancel: the wrapped `Cancel` sets `cancelled`, then the group kill hits `-pid`. `systemd-run` has exec'd, so pid and pgid are the test's. It returns `killed=false`. Correct.
- `proc.runMemoryCapped`, scope path, `systemd-run` cannot start the scope (no user bus in `cmd.Env`, or the unit is refused): `systemd-run` exits 1, so the call returns `killed=false` with an `ExitError`. That is misreported as a test failure. **Finding 1.**
- `proc.probeMemoryScope`, `systemd-run` missing, failing, a listing with no `0::` line, or `memory` absent from `subtree_control`: each returns `memoryScope{}`. Correct. A user manager that never answers: `.Output()` has no bound, so it blocks for ever. **Finding 2.**
- `proc.runDataLimited`, the process exited before `Prlimit` (`ESRCH`): it waits, which is correct. A `Prlimit` EPERM: it kills, waits and returns a wrapped error, so `RunAlone` returns an error. Correct.
- `proc.readMemoryLevel`, high `max` with max `8000`: the limit is 8000. Both `max`: skipped. Numeric high with no `memory.current`: skipped. The root has no files: skipped. Correct.
- `proc.fileCacheBytes`, an absent `memory.stat`: 0. A malformed value: skipped. Correct.
- `proc.readMemoryLimitAt`, an absent cgroup file: not found. The tightest headroom across levels wins. Correct.
- `proc.cgroupV2Path`, a v1 line followed by a `0::` line: returns the `0::` path. Correct.
- Windows `runMemoryCappedPlatform`, a child the test starts between `cmd.Start` and `assignProcessToJob`: the child is outside the job. **Finding 3.** An assignment failure kills, waits and returns the error. Correct.
- `createKillOnCloseJob` with `memoryLimit` 0: no memory flag is set, so the existing callers are unchanged. Correct.
- `testcover.RunAlone`:
  - A timeout plus an OOM line: TimedOut wins. Correct.
  - A kill with `MemoryLimit` 0: `capped` is false. Correct.
  - Under `RLIMIT_DATA`, the line `fatal error: out of memory` matches `outOfMemoryLine` and returns `MemoryCapped`. Correct.
  - A parent `Ctx` cancel: `killed=false` and no OOM line, so it is a plain failure, which the callers treat as cancelled. Correct.
- `testcover.ProfileBlocks`, a capped run: the non-nil `ExitError` path reports a failure. Correct.
- `gatecli.clampParallelToMemory`:
  - parallel 4, headroom 2.5 caps: 2. Correct.
  - Negative headroom: 1. Correct.
  - `!coversTree`: 1. Correct.
  - `limit` 0 would divide by zero, but `LoadConfig` refuses values below 1 and `&lt;&lt;20` keeps it above 0. Correct.
- `gatecli.memoryCappedParallel`, no limit found: the zero `MemoryLimit` has headroom 0 and `limited=false`, so it returns parallel. Correct.
- `gatecli.holdSlot`, both the inherited-slot and the acquired-slot paths: `sideBySide` is set before return. Correct.
- `gatecli.runMutants`:
  - A capped baseline is not rerun, gets no `timeouts` entry and sets `FailsUnmutated`. Correct.
  - A capped candidate run is recorded as a kill and listed in `capped`. Correct.
  - A queued covering job has its capped and timedOut entries cleared before the rerun, then `record` re-marks them. Correct.
  - A timed-out side-by-side covering run is rerun alone, and `TimedOut` reflects the alone result. Correct.
- `gatecli.runJobs`, a capped failure with `sideBySide` 2: no rerun alone. With `sideBySide` 1: no rerun. Correct.
- `gatecli.stopsOf`, an empty map: an empty slice, which stays `[]` in JSON because the stats are pre-initialised. Ordered by mutant, then test. Correct.
- `gatecli.runJob`, a baseline job with mutant 0 absent from `c.mutants`: nothing is logged. A mutant job: logged before the run. Correct.
- `gatecli.redundancyRun.runTest`, a capped run: returns the `reasonMemoryCapped` text with the MB value. Correct.
- `gateslot` `RedundancyTestMemory`, 2048: 2048&lt;&lt;20 bytes. Correct.
- `cmd/testtiming` `measurePackage`, a capped or timed-out run: `ActionFail` with no error, so it is classified as failed. Correct.

**Findings**

1. MEDIUM, design. `internal/proc/memorycap_linux.go:175-189`, together with `internal/proc/memorycap.go:68` and `internal/testcover/runner.go` (the `RunAlone` ExitError path).
   - A scope the host cannot start comes back as a failing test, not as an error. Causes include no user bus in the run's `cmd.Env` (the probe used the process env), a refused transient unit, or a user manager going away after the probe.
   - `systemd-run`'s own exit 1 is an `*exec.ExitError`, so `RunAlone` returns `ActionFail`.
   - In `lym gate redundant` that silently marks every baseline `FailsUnmutated`, or counts every mutant run as a kill.
   - This contradicts the `RunMemoryCapped` doc line "or reports a cap that could not be set, in which case cmd does not run on".
2. MEDIUM, design. `internal/proc/memorycap_linux.go:134`.
   - The probe calls `exec.Command(starter, ...).Output()` with no context or timeout.
   - A user manager that does not answer stalls the first capped run and `memoryCappedParallel` (`redundantmutants.go:508`) for ever, before any test bound applies.
   - That is likeliest under the slice-wide `memory.high` throttle this change exists for.
3. LOW, design. `internal/proc/memorycap_windows.go:392-397`.
   - The command is assigned to the memory-limited job only after `cmd.Start`, with no `CREATE_SUSPENDED`.
   - A process the test starts in that window is outside both the job memory limit and the `TerminateJobObject` cancel.

**Tool calls**

- `sed` over `internal/testcover/runner.go` (the `RunAlone` body), `grep` for `readIntFile`, and `grep` for the Linux `configurePlatformGroupKill`: the context had only the diff hunks, not the full `RunAlone` error flow or these two helpers.
- `sed` over `internal/gatecli/redundantmutants.go` (`holdSlot` and `runSideBySide`): needed to confirm `sideBySide` is set on every `holdSlot` path, since a zero value would block `runSideBySide`.
