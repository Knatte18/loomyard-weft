```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's changed functions and branches.

**Per-function list (case run → verdict)**

- `enterSession`, error returns before the goroutines start (`resolveWatchedSession`, `ReedGeometry` and `LoadConfig` each fail): these return `watchedSession{}` with nil `cancel` and nil `running` (`internal/reedcli/watchdog.go:336-349`). Both test callers `t.Fatalf` before they register `stopWatchedSession`, and the daemon `continue`s without storing the entry (`watchdog.go:607-611`). Nothing calls `Wait` on a nil `running`. **Correct.**
- `enterSession`, the `watchdog: off` branch (`cfg.Watchdog = "off"`, the `TestEnterSession_DrivesTheToldTmux` input): one `running.Go` wraps `watchPaneOOMKills`, and `running` is returned in the value (`watchdog.go:354-365`). `WaitGroup.Go` exists from Go 1.25, and go.mod declares 1.26. Its `Add` happens before the return, so a later `Wait` cannot miss it. **Correct.**
- `enterSession`, the enabled branch (`cfg.Watchdog = ""` or any other value): a second `running.Go` wraps `WatchNames` (`watchdog.go:367-372`). `watchNames` returns on `ctx.Done()` after the pass in flight (`internal/reedengine/namerepair.go:102-115`), and `repairNames` runs its tmux calls synchronously. **Correct.**
- `watchPaneOOMKills` after cancel, where ctx is cancelled during the first `reportPaneOOMKills`: the read in flight finishes, a new timer is made, and `select` sees `ctx.Done()` and returns (`internal/reedcli/oomnotice.go:152-161`). At most one read follows the cancel, and none follows `Wait`'s return. The told tmux path does not exist, so the read in `TestEnterSession` fails fast at the `has-session` exec. The `.lym` MkdirAll and lock file are already written by then (`internal/reedengine/lock.go:149-161`), and the wait covers that write. Everything the read calls is synchronous: `Status`, the tmux calls, the `proc` reads, `NotifyAsker`. **Correct.**
- `watchedSession` copied by value (daemon `known` map, test locals): `running` is a pointer, so every copy shares one WaitGroup and none copies a `sync.WaitGroup` value. **Correct.**
- `watchedSession` built without `running` (`internal/reedcli/watchdog_test.go:46-73` literals in `TestPlanSessionDiff`): `planSessionDiff` reads only the keys, and no production path reads `running`. **Correct.**
- Daemon cancel sites (`watchdog.go:482`, `596`, `621`): unchanged and only cancel. Nothing waits on `running`, so no stall is added. **Correct** as the summary states.
- `stopWatchedSession`, normal case (goroutines return): cancel, then `Wait` in a helper goroutine, then the `select` receives `stopped`. **Correct.**
- `stopWatchedSession`, boundary case (a goroutine never returns, e.g. a read blocked on a held op lock): after 30s it calls `t.Error`, which is legal inside a Cleanup. The helper goroutine leaks until the binary exits, which a reported failure makes acceptable. **Correct.**
- `stopWatchedSession` cleanup order in `TestEnterSession_DrivesTheToldTmux`: it is registered after both `TempDir`s and `SeedConfig` (`internal/reedcli/entersession_integration_test.go:25-49`), so LIFO runs it first. **Correct.**
- `stopWatchedSession` cleanup order in the `ReEntryReReadsFlippedConfig` subtest: it is registered after `offEng.Down` (`internal/reedcli/watchdog_integration_test.go:500, 515`), so it runs before Down. The parent's TempDir removal runs after the subtest's cleanups. The OOM watch reads a live session there, with no other op holding the lock, so the wait ends promptly. **Correct.**

**Findings**

None. Every path I ran gives the intended result, and no error is swallowed or state left inconsistent.

Outside my lens, for the handler: the daemon loop's deferred teardown (`watchdog.go:480-488`) still only cancels its known entries. `TestWatchdogDaemon` and `TestWatchdogReap` run `runWatchdogLoop` in process against real hubs, so their OOM watches can outlive `&lt;-loopDone`. Only the engines' `Down` cleanups serialize them, through the op lock.

Tool calls: none.
