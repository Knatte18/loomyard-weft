```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes, step by step on concrete inputs.

**Per function or branch: case run, verdict**
- `memoryGuard.run`:
  - Sampler error, then a sample over the cap: continues, then kills. Correct.
  - `ErrUnsupported`: logs once and returns. Correct.
  - ctx cancelled between samples: returns the lines so far. Correct.
- `testTrees`:
  - Empty tree: returns nil. Correct.
  - A `.test` nested under a `.test`: the outer tree's pids and held memory include the inner's. Correct.
- `kills`:
  - Inner tree at 5000 MB under an outer tree at 5100 MB: only the inner is killed. Correct.
  - Tree held exactly at the limit (4096 MB): not killed, since the check is `>`. Correct.
  - Pressure check with `found=false`: nil. Correct.
  - Pressure with headroom 512 MB of a 16384 MB Max: below a tenth, so the largest tree of at least 1 GB is killed. Correct.
  - All trees under the 1 GB floor: nil. Correct.
- `killTree`:
  - `describe` fails because the process is gone: dir and filter are empty, and the line says the directory could not be read. Correct.
  - A kill error is logged at Debug and the line still says "killed". Acceptable.
- `memoryGuardApplies`:
  - Limit 0: off.
  - Enclosing pid "77" that is an ancestor: off.
  - Enclosing pid "78" that is no ancestor: on.
  - Empty or unparsable enclosing value: on.
  - All correct.
- `guardOwnTests`: when the guard does not apply, it returns a no-op. When it does, stop cancels and waits; the `defer` order in `runTestMap` stops the guard before `run.close()`. Correct.
- `runTiedChild`, `memoryLimit=0`: the old path, unchanged. Correct.
- `runTiedChild`, `memoryLimit>0`, scope starts and the wait returns: `capKilled` is the tree kill. Correct apart from **F1**.
- `runTiedChild`, `memoryLimit>0`, the scope never ran the command: the error is no `ExitError`. See **F2**.
- `runTiedChild`, `memoryLimit>0`, `os.Pipe` fails or a Windows job cannot be created: `cmd.Process` stays nil, so `runInvocations` refuses with "could not start the shell ... put the platform shell on PATH". The way forward is wrong (**F3**, LOW).
- `runInvocations`:
  - Guard, process cap and watchdog goroutines write distinct fields of `stops` and are joined by `watchdogDone.Wait` before the read. No race.
  - `stops` is reset in practice, since the loop continues only when `stopped()` is false. Correct.
- `runTier`:
  - Guard lines are printed live and added to the log and causes. The cap line is printed once and added to the log and causes. Exit is forced to 1. Correct.
  - `memoryStopped` blocks the shortage rerun.
- `runStops.memoryLines` and `runStops.stopped`: with empty input they return nil and false. Correct.
- `failedOnShortage` with `memoryStopped`: returns "". Correct.
- `decodeTier`:
  - `memory_mb` absent: 0, uncapped.
  - `memory_mb: 0`: refused.
  - `memory_mb: 8192`: set.
  - All correct.
- `decodeConfig`: `memory_quit_mb: -1` is refused and 0 is accepted. Correct.
- `Config.MemoryQuit` and `Tier.MemoryLimit`: 0 gives 0, and 4096 gives 4096<<20. Correct.
- `TreeSample`:
  - Stat with fewer than 22 fields: skipped.
  - RSS is pages × page size.
  - Parent is set.
  - All correct.
- `ReadProcess` and `ListProcesses`: a gone pid returns false; behaviour is the same as before. Correct.
- `runMemoryCapped`, Linux:
  - Data-limit path: returns false, false, and the wait error.
  - Cancelled: false, false.
  - OOM kill of a child: killed=false, treeKilled=true.
  - Command SIGKILLed by the kernel: true, true.
  - `RunMemoryCapped` still returns `killed`, so its contract is unchanged.
  - All correct apart from **F1**.
- `runDataLimited` with a caller start: when Prlimit fails, it kills and reaps through `wait()`, so there is no double `cmd.Wait`. Correct.
- `runMemoryCappedPlatform`, Windows, with the tied start: the tied handle's goroutine owns `cmd.Wait`, and `wait()` is `handle.Wait`, so there is no double wait. When `assignProcessToJob` fails, it kills and then waits. Correct.
- Guard kill inside `RunAlone`'s scope (`lym gate redundant` and `lym gate testmap`): misreported as a cap kill. See **F4**.

**Findings**

**F1. MEDIUM, design.** `internal/proc/memorycap_linux.go:160-168` and `internal/gatecli/tierrun.go:404-406`.
- Under a tier's `memory_mb`, a SIGKILL of the tier shell from the process cap is reported as a `memory_mb` kill.
- `processCap.run` SIGKILLs every pid carrying the run marker (`internal/gatecli/processcap.go:43,75-80`).
- The shell carries that marker, because `cmd.Env = env` holds `LYM_GATE_TEST_PID` (`run.go`, `exe.env`).
- `runMemoryCapped` then sees the shell signalled with SIGKILL and no cancel, so it sets `killed=true` and returns `treeKilled=true`.
- `runInvocations` sets `stops.memoryCap`, and the tier prints both the process-cap line and a false "held more than its memory_mb ... the kernel killed" line.
- Any outside SIGKILL of the shell is misreported the same way.

**F2. MEDIUM, design.** `internal/gatecli/child.go:64-79` and `internal/gatecli/tierrun.go:407-414`.
- When a tier's `memory_mb` scope cannot run the command, `runMemoryCapped` returns a non-`ExitError` error (`memorycap_linux.go:156-158`).
- `tierExitCodeOf` maps it to 1, and `cmd.Process` is non-nil, so no refusal is raised.
- The tier fails with no cause line, no stderr line and no way forward. The error reaches only the Info log "tier command ended".

**F3. LOW, design.** `internal/gatecli/tierrun.go:409-411` with `internal/gatecli/child.go:64-65`.
- A capped start that fails before the shell starts sends the user the wrong way forward.
- Examples are a failed `os.Pipe` (`memorycap_linux.go:129-131`) or a Windows job that cannot be created (`memorycap_windows.go:15-18`).
- Both leave `cmd.Process` nil, so the refusal says to make the dir existing and put the shell on PATH, and it omits the real cause.

**F4. MEDIUM, design.** `internal/gatecli/memoryguard.go:376-380` with `internal/testcover/runner.go:178-186`, `internal/proc/memorycap_linux.go:160-161` and `internal/gatecli/redundantmutants.go:234-245,397`.
- In `lym gate redundant` and `lym gate testmap` with a capping scope, the test binary is the scope command's own process (`systemd-run` → `sh` → `exec` bin).
- A guard SIGKILL of it reads as `killed=true`, so `RunAlone` returns `MemoryCapped`. The run is reported "ended at the memory cap of 2048 MB" (`redundancy.go:59`) instead of the guard's limit or pressure.
- In the mutation check every failed run counts as a kill, and a `MemoryCapped` run is never re-run alone (`redundantmutants.go:397`).
- So a pressure-trigger kill, which happens because memory is short elsewhere on the machine, counts as a cover's mutant kill. It can falsely confirm a candidate as redundant, or set `FailsUnmutated` on an unmutated run.

Tool calls:
1. `tierrun.go` `printLine` and `tied_windows.go` `startPlatform`: needed for nil-writer safety and the Windows double-wait question; neither was in context.
2. `redundantmutants.go` lines 228-250 and 390-400: needed to see how a `MemoryCapped` or failed mutant run is counted; that code was not in context.
