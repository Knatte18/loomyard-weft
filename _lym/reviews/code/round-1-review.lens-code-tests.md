```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that would fail if it broke, or none.

**Per behaviour, the test that pins it**

| Behaviour | Test that pins it |
|---|---|
| `proc.Reap` waits on a started command in the background | `TestReap` (`internal/proc/proc_linux_test.go`). It fails if the wait goroutine goes, because the pid stays alive as a zombie. |
| `proc.ReapInheritedChildren` returns and waits on exited and running children | `TestReapInheritedChildren`. It checks the returned pids and that both children are reaped. |
| `ReapInheritedChildren` returns no pids and no error where listing is unsupported (`errors.ErrUnsupported`) | None. It is a Windows-only path, and the Linux-only test file cannot reach it. |
| `runproc` listing reads a zombie as gone (`lister.live`) | `TestRegisterAndList` zombie case (`internal/runproc/record_test.go:146-162`). It runs through the `isRunning` seam of a hand-built `lister`. |
| `List` and `Owners` use `proc.IsRunning` in production | None. A swap back to `proc.IsAlive` in `internal/runproc/record.go` fails no test, because no test lists a real zombie through `List` or `Owners`. |
| The `lister` empty-start-time rule (`live`: `record.StartTime == ""` once the pid runs) | The Windows-branch case in `TestRegisterAndList` (`record_test.go:134-144`). |
| End keeps a zombie leader's record listed so the group signal reaches its members | The existing end-sequence row "a group whose leader is gone still reports a member that survives SIGKILL" (`internal/runproc/end_test.go:190-198`). It expects term and kill signals to group 12, and the fake's `isAlive` counts a zombie entry as alive. |
| `productionEnder` wires `isAlive: proc.IsAlive` and `end` builds the `lister` from it | None. The `ender` struct has no running-versus-alive seam. The diff's net change to `end_test.go` is zero, so the row above is not new coverage. |
| `runDobbyWith` calls `reap` after `placeDobby` succeeds and before the dobby starts anything | None. The only caller in tests, `TestRunDobbyWith_TracesEachExitBeforeRun` (`internal/beamcli/dobbytrace_integration_test.go:59`), passes `func() {}`. No test asserts that `reap` runs on a placed start, that it does not run on a refused placement, or that it runs before `selfexec` and the lock. The commit title "a refused dobby start reaps no test child" has no row. |
| `runDobby` wires `reapInheritedChildren` (log lines for the error and for non-empty pids) | None. |
| `lym orch watch` `RunE` calls `proc.ReapInheritedChildren` before it starts anything | None. |
| Each detached spawn calls `proc.Reap`: board sync, detached push, front-task start, `dobbyengine.SpawnDetached`, reed watchdog, reed remove, battencli pass, adopt check, termwindow `OpenAttach` | None per site. Most of them refuse to spawn under `testing.Testing()` or have no test seam on the real start. Only `proc.Reap` itself is pinned. |
| `dobbyengine.SpawnDetached` start closure: `Start`, then `Reap`, with no `Reap` on a `Start` error | None. `spawnDetached` takes the start seam, but the closure that adds `Reap` sits above it, behind `ErrSpawnUnderTest`. |
| `SpawnWatchdog` now returns early on a `Start` error, so `Reap` never sees an unstarted cmd | None. |
| `fronttask.startDetached` returns the pid without `Process.Release()` | None. It is guarded by `testing.Testing()`. |

**Findings**

1. MEDIUM, `design`. `internal/beamcli/cli.go:386-416` and `internal/orchcli/watch.go:98-103`.
   - What is unpinned: the placement of the startup reap in both daemons, and the log branch of `reapInheritedChildren`. This is the behaviour that fixes the reported zombie pile. The ordering matters because a reap after the process starts its own child races that child's own wait.
   - The only `runDobbyWith` test substitutes a no-op, so deleting the call or moving it after the first spawn fails nothing. There is no row for `orch watch` at all.
   - Evidence: `dobbytrace_integration_test.go:59`; no `watch` test touches `ReapInheritedChildren`.
   - A test could pass a recording `reap` and assert it runs once after placement and never on a refused placement. The watch case would need its own seam.

2. LOW, `design`. `internal/runproc/record.go:121-127` and `internal/runproc/end.go:137-141`.
   - What is unpinned: the production wiring. `List` and `Owners` use `proc.IsRunning`, and `productionEnder` keeps `proc.IsAlive` for End. Both are covered only through fake seams, so swapping either function fails nothing.
   - A cheap guard is a Linux test that registers a real zombie child and asserts `List` prunes it.
   - The commit message says End deliberately keeps zombie-leader records, but no test names or isolates that choice.

3. LOW, `design`. The `proc.Reap` call sites listed in the table (`internal/boardengine/spawn.go:57` and the rest).
   - Nothing pins that each detached spawn calls `proc.Reap`.
   - The rule is a PATTERN entry without a `(test)` marker, so enforcement is review only. I raise it so the handler can decide whether the sites that can be reached count as a coverage gap.
   - The most reachable candidate is the `dobbyengine.SpawnDetached` closure.

**Test quality, no finding**

- `TestReapInheritedChildren` is correctly non-parallel and says why. It polls with a deadline, not a fixed sleep, and it cleans up the `sleep 30` child.
- The zombie case is a case in `TestRegisterAndList`, not a new test, so it is no duplicate.
- `TestReap` and `TestReapInheritedChildren` overlap only in using `awaitReaped`, not in coverage.

**Tool calls:** none. The manifest already held the diff, the test files and the `end_test.go` fake.
