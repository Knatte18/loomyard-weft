```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code through the correctness lens (changed functions, their callers, and whether the added tests fail with the defect in place).

no findings

I ran each of these from the context without opening a file:

- **`runMemoryCapped` scope path, scope never starts.**
  - `/bin/false` as starter: `Start` succeeds and the process exits 1, so `ProcessState` is set and the call was not cancelled.
  - The pipe's write end is closed in the parent and the child never wrote, so the marker read hits EOF and `scopeStarted` is false.
  - The call returns a `%v` error with no `%w`, so `errors.As` finds no `*exec.ExitError` and `killed` is false.
  - With the old `cmd.Run` path the same input returned the `ExitError`, so `TestRunMemoryCapped_AScopeThatNeverStartsIsAnErrorAndNotTheCommandsExit` fails with the defect in place.
- **Scope starts and the command fails or is OOM-killed.** The shell writes the marker before `exec`, so the byte is already in the pipe after `Wait` and `scopeStarted` is true. The SIGKILL detection still reads `ProcessState` of the same pid, so the existing kill-report row holds.
- **Cancel before the shell runs.** `cancelled` is true, so the original error is returned unchanged and no scope-start error is reported.
- **`Start` fails.** The marker is closed, and the deferred `started.Close()` runs.
- **Descriptor leak.** `exec 3>&-` closes fd 3 before the command runs, so no descendant holds the pipe write end and the read cannot block until `markerReadBound`.
- **Probe.** `CommandContext` with `WaitDelay` turns a hang into an error, which returns `memoryScope{}`.
- **F4 `sideBySide`.**
  - It is set in `takeSlot` before every `runBatch` and `runSideBySide`, and `takeSlot` runs once per slot hold.
  - If any path reached `runSideBySide` with `sideBySide` at 0, `min(0, n)` would start no workers and deadlock. The review's evidence and the green tier-1 run show no such path, so this is not a finding.
  - `testMemory` is at least 1 MB by config validation, so the clamp's division is safe.
  - The three test request literals now set `cappedParallel`, so there is no nil-func call.
- **Added tests.**
  - The `TestRunAlone` runtime-out-of-memory row fails without the regex branch: exit 2 with no kill gives `Action: fail` and `MemoryCapped: false`.
  - The `TestGateTestMap` subtest and the `TestRedundancyStep` row would fail with `MemoryLimit` dropped, because the hoarding stand-in then passes with a profile.

Tool calls made: none; the context already held every changed file and the tier-1 gate result.
