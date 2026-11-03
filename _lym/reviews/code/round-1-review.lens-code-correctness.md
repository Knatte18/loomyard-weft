```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the `19675459` diff in `internal/gatecli`, running each changed function or branch with a concrete case.

**Per-branch list**

| Function or branch | Case run | Verdict |
|---|---|---|
| `runTiedChild`, `cmd.WaitDelay = pipeWaitDelay` | Production 10s; helper 100ms; a zero delay, as a bare `testRequest{}` would pass | Correct. Zero means unbounded, and every real constructor passes the default (`cli.go:90`, `rungate.go:28`, helper `durations`). |
| `runTiedChild`, `StartTied` failure | Shell cannot start | Correct. Returns the error before `killLeftovers`; `cmd.Process` stays nil and the `build.go:50` and `tierrun.go:242` refusals still fire. |
| Clean zero exit, no leftovers | `Wait` returns nil | Correct. `killLeftovers` scans `/proc`, finds no member of the group, returns `(false, nil)` and logs nothing. `waitErr` nil is returned. |
| Zero exit with an orphan holding the pipe | `Wait` returns `exec.ErrWaitDelay` after the delay | Correct. `exec.Cmd.Wait` reports `ErrWaitDelay` only when the exit was clean and the context had no error. The branch returns nil, so `exitCodeOf` gives 0 and the tier or build passes on its own output. The Warn log is a single event, not in a loop. |
| Non-zero exit with an orphan (the 134 case) | `Wait` returns `*exec.ExitError` | Correct. `ErrWaitDelay` is not reported over an `ExitError`, so the branch is skipped. `exitCodeOf` gives 134, the tier records outcome `failed` and `TimedOut` false, and `runInvocations` stops at the first non-zero exit. |
| Timeout or cancelled context | `Cancel` kills the group and the shell dies by signal | Correct. `ExitError` with `ExitCode() == -1` gives exit 1, as before. `timedOut` in `tierrun.go:88` and `build.go:54` still sees `exit != 0`. `killLeftovers` finds the group already empty and does nothing. |
| Zero exit with the context cancelled during the wait | Context error beats `ErrWaitDelay` in `Wait` | Correct. The error is returned unchanged, so a cancelled run is not turned into a pass. |
| Timeout expiring inside the post-exit delay window | The context watcher has stopped after `Process.Wait` | Correct. The exit is 0 and `timedOut` requires `exit != 0`, so the run passes. This matches the old behaviour for a shell that had already exited. |
| `killLeftovers`, Linux, orphan in the group | Orphan's `pgid == rec.PGID` and its start time is at or after the shell's | Correct. A live member keeps the group id from being reassigned, so the match is the recorded tree. `SIGKILL` goes to `-PGID`, and `ESRCH` is mapped to `(false, nil)`. |
| `killLeftovers`, Linux, pid reuse (the focus question) | Shell reaped, group empty, and the pid reused by a new process that is its own group leader | Not reachable in practice. It needs a full pid-space wrap between `cmd.Wait` returning and the `/proc` scan, and a new group leader with that exact pid. When an orphan holds the pipe, a live member pins the id for the whole delay. A reused pid that is not a group leader has `pgid != pid` and cannot match. The residual window is the empty-group case, and it is bounded by microseconds. No finding. |
| `killLeftovers`, `StartTime` unreadable at start | `rec.StartTime == ""` | LOW, finding 1 below. |
| `killLeftovers`, `KillTree` returns an error | `/proc` unreadable | Correct. Warn is logged, the run result is unaffected, and nothing is retried. |
| `killLeftovers`, Windows | `killTreePlatform` returns `(false, nil)` | Correct, and the doc says so. The job closes in `Tied.Wait` through `releasePlatform` after the delay, which ends the tree. |
| Ordering: kill after the delay, pipe read already cut | Orphan holds the pipe | Correct but slow. `Wait` returns after the delay, which closes the read end and ends the copy goroutines. The group kill then runs, so no leftover holds a gate slot or the pipe past `runTiedChild`'s return. The cost is the full delay (10s) per orphan-holding exit before the kill. That is a latency choice, not a defect. |
| Kill coverage on timeout, cancel and failed start | See rows above | Covered. Timeout and cancel are killed by `Cancel` through `Tied.Kill`, and `killLeftovers` then sweeps any stragglers. A failed start has no process, so nothing is left. |
| `newCommand` and `testRequest` plumbing | Production, `runnerCmd` re-entry, `RunGate`, `detachrun_test.go` | Correct. `pipeWaitDelay` is passed on every path, including the `runnerCmd` rebuild at `cli.go:121`. |
| `build.go` path | Build shell exits 0 or non-zero with an orphan | Correct. It goes through the same `runTiedChild`. The `timedOut` and `exit` handling is unchanged, and `ErrWaitDelay` becomes exit 0 and a passed build. |

**Findings**

1. LOW, `design`, `internal/proc/tied_linux.go:31` and `:61-64`, reached from `internal/gatecli/child.go:56`.
   - When `StartTime(pid)` fails at start, the record's `StartTime` is `""`.
   - `killTreePlatform` then returns `(false, nil)` and `killLeftovers` logs nothing, so the leftover sweep is skipped without a trace.
   - The tier still ends in bounded time because of `WaitDelay`, but an orphan keeps running inside the slots with no log line.
   - The condition is rare, since `/proc/<pid>/stat` of the shell is normally readable, so this is a silent-degradation note rather than a defect in the common path.

**Tool calls:** none. The manifest already held every definition this lens needed.
