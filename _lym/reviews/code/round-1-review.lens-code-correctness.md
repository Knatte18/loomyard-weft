```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes.

**List: each function or branch, the case run, and the result**

- `waitForFixtureProcessesToExit`, root resolution. Case: `EvalSymlinks` succeeds on the resolved lymtemp dir; case: it fails. The first gives the resolved root. The second calls `t.Fatalf` on the test goroutine before any cleanup is registered. Correct.
- `waitForFixtureProcessesToExit`, cleanup order. Case: inside `newBadReedUpFixture`, after `hubforge.CopyRegistered`. The copy's `TempDir` removal and its junction-removal cleanup are registered first and the wait last, so the wait runs first. In `waitForFixtureProcessesToExit` itself, `t.TempDir()` is called before `t.Cleanup`, so removal still runs after the wait. Correct.
- Loop, empty scan on the first poll. Case: no process under root. `processesUnder` returns nil, so `len == 0` and it returns. Correct.
- Loop, process exits within patience. Case: a board sync that exits at 3 s with patience 10 s. Polls continue while `Before(deadline)` holds, then a scan comes back empty and it returns. Correct.
- Loop, zero or short patience. Case: `patience = 0`. The deadline has passed at the first scan, so `left` is killed at once and the deadline resets to now+0. The next poll, 20 ms later, either returns or fails. Correct for its contract.
- Loop, the kill branch. Case: a sync pid that is still running; `KillPID` returns an error that is ignored. It is killed by SIGKILL. The process turns zombie until it is reaped, and a zombie's `/proc/&lt;pid&gt;/cwd` cannot be read, so `ProcCwd` gives false and the zombie drops out of the scan. A `KillPID` error on a pid that is already gone is harmless to ignore. Correct.
- Loop, a pid that first appears under root after `killed` is set. Case: the sync forks a git child between the scan and the kill, or a killed process's child goes on to start another process. The new pid is never killed. If it lives past the second patience, `t.Errorf` reports it as "still run … after they were killed" and the cleanup returns with it still running. **Not correct** (F1).
- Loop, a killed pid that is still running after the second patience. Case: a process stuck in D state. `Errorf` reports it and the cleanup returns, and `tmuxkit.Main`'s sweep then handles it. The message matches the doc comment. Correct.
- `ProcArgv` failure in the log line. Case: the process is gone between the scan and the read. `argv` is nil and the log shows `[]`. Correct.
- `processesUnder`. Case: a process whose cwd is root itself. It is excluded by the separator suffix; nothing a fixture spawns runs at the bare root, because hub copies sit in numbered subdirs. Case: another user's pid whose cwd cannot be read. It is skipped. Case: the test binary's own pid. It is excluded. Correct.
- `TestWaitForFixtureProcessesToExit_KillsAProcessThatOutlivesItsPatience`, case: the cleanup kills the process. The subtest's cleanup kills `sleep` after 100 ms, `cmd.Wait` returns an `ExitError` whose `WaitStatus` shows SIGKILL, and the test passes.
- Same test, case: the kill is removed. The second patience ends with `Errorf`, so the subtest fails, and `select` then times out at 5 s with `Fatal`. The test fails. Correct.
- Same test, case: `cmd.Start` fails. `Fatalf` fires inside the subtest and the cleanup finds nothing. The parent then sees a non-nil `cmd` whose `Process` is nil and calls `Fatal`. That second failure is redundant but correct.
- Same test, type assertion. Case: `exitErr.Sys().(syscall.WaitStatus)`. The file only compiles on Linux, where `Sys()` is a `WaitStatus`, so the assertion is safe. Correct.
- `newBadReedUpFixture`, registration. Case: the "go driver" subtest with nested subtests that are not parallel. The hub root belongs to the "go driver" subtest's temp root, and the verbs' detached sync has its cwd in the copy's board dir under that root. The cleanup runs after the nested subtests finish, so no verb is still running then. Correct.
- `TestLoomResumeReconcilesTheHubConfigBeforeArming` drops its explicit call. Case: the fixture now registers the wait. Only one wait is registered, so it is covered once. Correct.

**Findings**

- **F1, LOW, design.** Location: `internal/loomcli/smoke_helpers_test.go`, the `waitForFixtureProcessesToExit` cleanup loop (diff lines 133–144).
  Summary: after `killed` is set, a pid that first appears under root is never killed. After the second patience it is reported as "still run %s after they were killed" and left running. That contradicts the doc comment's "ends every process" and misreports a process that was never killed as having survived a kill.
  Trigger: a detached `lym board sync` forks a git child between the scan and `proc.KillPID`. Killing only the sync's pid leaves that child running, and the child's cwd is the board dir.
  Fix direction: kill every pid in `left` on each scan past the deadline, keeping track of which pids were already killed, so the failure names only pids that were killed and survived. Alternatively, word the message to cover pids that were never killed.
  Evidence: diff `001-internal-loomcli.diff` lines 124–145, and `internal/boardengine/spawn.go` (`cmd.Dir = abs`, a detached own-session process).

I made no tool calls.
