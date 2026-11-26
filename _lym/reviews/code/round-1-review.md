---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedcli/entersession_integration_test.go#TestEnterSession_DrivesTheToldTmux
    summary: No assertion depends on the wait the fix adds, so TestEnterSession_DrivesTheToldTmux passes unchanged if enterSession goes back to a bare go statement for the OOM watch
    fix: "In TestEnterSession_DrivesTheToldTmux, before the enterSession call, assert that filepath.Join(location.AnchorPath(), \".lym\", \"reed.lock\") does not exist (os.Stat returns an error satisfying errors.Is(err, fs.ErrNotExist)). After the existing TmuxPath and repairsNames checks, call stopWatchedSession(t, ws) in the test body, then assert that the same path exists. Build the path from the literal \".lym\" and \"reed.lock\" segments, never from an engine method. Keep the t.Cleanup registration; a second cancel and Wait are harmless, and the cleanup still covers the t.Fatalf paths."
---

### [LOW] The cleanup wait has no assertion that fails without it

**Location:** `internal/reedcli/entersession_integration_test.go:45-56`

**Issue:** The fix makes `running` count the OOM watch so `stopWatchedSession` blocks until that goroutine returns.
Nothing in either test that calls `stopWatchedSession` observes that it blocked.
If `enterSession` started the OOM watch with a bare `go` again, `running.Wait()` would return at once and both tests would still pass.
The behaviour has an observable effect that a test can pin:
`watchPaneOOMKills` runs its first `reportPaneOOMKills` before it checks `ctx` (`internal/reedcli/oomnotice.go:152-153`).
That read goes through `Engine.Status` into `withOpLock`, which creates `<anchor>/.lym` and `reed.lock` before any tmux call (`internal/reedengine/lock.go:149-157`).
At the `.` anchor that is the prime worktree, the `.../warp-bare` directory the issue's failure names.
Once the wait has returned, the lock file therefore always exists.
Without the count, it is usually still missing when the wait returns.
Asserting it absent before the entry and present after the wait gives the fix a check that fails, most of the time, when the count is dropped.

## Holistic review

**Root cause.** The issue's failing path, `.../container/warp-bare-LYMHUB/warp-bare`, is the hub's prime worktree (`internal/hubforge/hub.go:226-229`, at anchor `.`).
In `TestEnterSession_DrivesTheToldTmux` the only goroutine `enterSession` starts is the OOM watch, since the worktree is seeded `watchdog: off`.
Its first strand read writes the `.lym` directory and `reed.lock` under that worktree whatever the told tmux binary is,
because the op lock's pre-flight checks the geometry and not the binary's presence.
The commit's `Root-Cause:` trailer names that defect: no way to await the goroutines' exit, so the cleanup raced the first read's lock creation.
It names a defect, not the symptom, and the diff removes it.

**Wait completeness.** `watchPaneOOMKills` and `watchNames` both run their I/O synchronously.
That covers the tmux calls, the `proc` reads, the journal read, `NotifyAsker`, `repairNames` and `pasteSubmitLocked`.
Neither starts a goroutine or a process it does not wait on, so each returns only after its last write.
`running.Go` registers each goroutine before `enterSession` returns, so no `Wait` can miss one.

**Cleanup ordering.** In `TestEnterSession_DrivesTheToldTmux`, the wait is registered after both `t.TempDir` calls and `SeedConfig`, so it runs first.
In `ReEntryReReadsFlippedConfig` it is registered after `offEng.Down`, so it runs before the session's teardown.
Both run before the parent test's temp directories are removed.

**Production.** The daemon's cancel sites (`internal/reedcli/watchdog.go:482`, `:596`, `:621`) are unchanged.
The error returns and the `TestPlanSessionDiff` literals build `watchedSession` with a nil `running`, which nothing waits on.
The summary's claims hold, including the `two-way` door: the diff changes no record format, config meaning or outward push, and its blast radius is reed.

**Other paths with the same race.** `TestWatchdogDaemon` and `TestWatchdogReap` run `runWatchdogLoop` in process against real hubs.
Its deferred teardown (`internal/reedcli/watchdog.go:480-488`) still only cancels the sessions the loop entered.
Their OOM watches can therefore still be mid-read when `<-loopDone` returns.
The engines' `Down` cleanups take the same op lock, which serializes an in-flight read before teardown, so the window there is narrower than the one this diff closes.
The board entry asks only for the test it names, and the commit states why the loop does not wait.
So this is no finding of this round: closing it would change the daemon's exit path, which the entry does not license.
A separate board entry is the way to take it up if those tests ever fail the same way.

## Rejected

- code-tests' MEDIUM on the missing assertion is kept as F1 at LOW: the gap is in a test-only wait over unchanged production behaviour, and the assertion it asks for detects the regression only probabilistically.
- code-tests' LOW on the name-repair goroutine's count having no test: when cancelled, that goroutine sits in its first 10s timer and writes nothing, so counting it has no observable effect a test can pin short of goroutine inspection.
