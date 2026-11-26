```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **`running` counts the OOM-watch goroutine, so `running.Wait()` returns only after `watchPaneOOMKills` has returned** (`internal/reedcli/watchdog.go:354-357`).
   None. `TestEnterSession_DrivesTheToldTmux` and `TestWatchdogDaemon/ReEntryReReadsFlippedConfig` call `stopWatchedSession`. Neither asserts anything that depends on the goroutine having stopped.
   If the line went back to a bare `go watchPaneOOMKills(...)`, `Wait` would return at once and both tests would still pass. This is the fix's whole behaviour.
2. **`running` counts the name-repair goroutine** (`internal/reedcli/watchdog.go:367-371`).
   None. Both tests enter a `watchdog: off` worktree, so this branch never runs under a wait. The mutation report's "5 uncovered changed blocks" agrees.
3. **`stopWatchedSession` fails the test when the goroutines outlive their cancel by 30s** (`internal/reedcli/entersession_integration_test.go:69-73`).
   None, and none is needed: this is a test helper's own failure branch, not a production behaviour.
4. **The `running` field is set on both successful returns of `enterSession`** (`watchdog.go:364`, `watchdog.go:372`).
   Pinned for the `off` return only: a nil `running` would panic in the two tests above. The enabled return (`watchdog.go:372`) has no test that waits on it, the same gap as behaviour 2.

**Findings:**

- **MEDIUM, design. The wait the fix adds has no assertion.** Location: `internal/reedcli/entersession_integration_test.go:45-56`.
  `TestEnterSession_DrivesTheToldTmux` passes whether or not `running` counts the OOM watch.
  - Evidence: `watchPaneOOMKills` (`oomnotice.go:149-163`) starts its first `reportPaneOOMKills` read before it checks `ctx`. That read runs `Engine.Status` (`lifecycle.go:1297-1299`) and then `withOpLock` (`lock.go:138-161`).
  - `validateToldTmuxIdentity` (`server.go:216-236`) does not check that the tmux binary exists. So with the told tmux path that does not exist, the read still reaches the `MkdirAll` of `&lt;anchor&gt;/.lym` and creates `reed.lock` (`lock.go:149-157`, `lifecycle.go:32-33`). A cancel does not interrupt this first read.
  - Suggested fix: once `stopWatchedSession` returns, assert that `filepath.Join(location.AnchorPath(), ".lym", "reed.lock")` exists. Take the expected path from a literal or from `lymdirs`, not from the engine. The file exists every time the wait holds, and it is very likely missing when the goroutine is not counted.
  - Without an assertion like this, the race fix is untested.
- **LOW, design. The name-repair goroutine's count has no test.** Location: `internal/reedcli/watchdog.go:367-371`.
  No test enters a worktree with the watchdog enabled and then waits on `running`. `TestWatchdogDaemon/ReEntryReReadsFlippedConfig` (`watchdog_integration_test.go:511-518`) seeds `off`.
  `stopWatchedSession` would pass even if this branch used a bare `go`. The repair loop sits in `t.After(10s)` (`namerepair.go:103-107`), so a cancelled loop returns quickly. A test could enter an enabled worktree and assert that `running.Wait()` returns within the helper's bound after the cancel. That proves the goroutine is counted and returns, though the bound only catches a goroutine that never returns.

Tool calls:
1. `grep` of `validateToldTmuxIdentity` and `stateDir` in `internal/reedengine`: I needed to know whether the op lock's pre-flight refuses a tmux binary that does not exist before it creates `.lym`. The context held only the call site in `lock.go`.
