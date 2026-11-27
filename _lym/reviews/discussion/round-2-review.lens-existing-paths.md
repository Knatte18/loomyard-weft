```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that meet the record's new state, files and settings.

**Paths traced, per decision**
- D1, the `termwindow.resolve` Konsole branch, which requires `DISPLAY`/`WAYLAND_DISPLAY` and `konsole` on PATH: survives, because the pre-launch checks sit on top of it.
- D1, a dobby started from a reed tmux server opening a window through the Open-Window row (`runRowCommand` with `commandEnv`, which strips only the strand name): survives, and the record already states this as D1's bound.
- D2, the three title sources: the Open-Window row's `open_window_command` default passes `--title {title}`, `lym machine start` passes `ConsoleWindowTitle`, and `lym reed window` uses `terminalWindowTitle`. Survives, since the hidden verb gets the title the window was actually given.
- D2 and D12, `OpenAttach` and `machinecli`'s `openTerminal` seam return only an error, with no pid: survives as a planned change and is not a finding.
- D3, the temp script under `lymtemp` against `lym`'s run-dir removal on exit and `Root.Sweep`: survives.
- D7, `dobbyengine`'s per-task key checks (`config.go:161`, `CheckLocalValues`) and the board's per-task key list (`boardcli/cli.go:587`): survives as a planned key addition.
- D9, the places file under `BoltDir(hub)/.lym` against `state.UpdateJSON`'s one exclusive lock across read, mutate and write: survives.
- D10, the Open-Window row's 30 s bound and the batten reopen's `openWindowTimeout`, both run through `runRowCommand` into a log file: survives, because the verb returns at once.
- D10, the per-user lock in `standalonestate.UserStateDir` beside `launchers.lock` and the machine record: survives, as long as the child creates the directory the way `desktoplauncher.acquireLock` does.
- D10, the flock held by a child that `runproc.End` kills: survives, because the OS releases it.
- D10, `runproc` registration "when the opening verb runs inside a run": **breaks**, see F-EP1.
- D12, `lym machine start`'s order (`runBeamStarts`, then `openWindows`, which opens windows only for worktrees with `startStarted`): survives, and the record now states it correctly.
- D12, `battencli.reopenWindow`, which runs only with no client attached, as on a fresh machine start: survives, and D8's parent wait covers it.
- D12, `lym machine start` running no `window_command` when the checks fail: survives, matching `openWindows`.
- D12, `lym ide`'s `desktophook.AfterWindow` path: survives, since it is untouched and out of scope.
- D12 with D10, `runproc.End`'s SIGTERM, 5 s grace (`endGrace`), then SIGKILL across three re-list passes, against a child that "runs `window_command` on every exit path": **breaks**, see F-EP2.

**Findings**

- **F-EP1, LOW, design, `_lym/discussion/decision-record.md:209` (D10)**
  - Summary: D10 registers the child in `runproc` "when the opening verb runs inside a run", but nothing tells `lym reed window` that it runs inside a run.
    The Open-Window row and the batten reopen start it through `runRowCommand`, whose `commandEnv` strips the strand-name variable.
    The verb gets only a directory and `--title`, so it has no owner slug to register under.
    An unregistered child outlives the run's teardown by up to about five minutes of waits.
    It then writes a places entry and runs `window_command` in a worktree that has been removed.
  - Evidence: `internal/battencli/land.go:204-233` (`runRowCommand`, `commandEnv`); `internal/battenshed/template.yaml:2` (`open_window_command` default); `internal/reedcli/window.go` (the verb's only inputs are the directory and `--title`).
  - Fix: in D10, name how the child learns its run owner, such as from the worktree's board slug resolved from the directory, or state as a bound that it is unregistered and what it does once its worktree is gone.
- **F-EP2, LOW, consistency, `_lym/discussion/decision-record.md:228` (D12) with `:208` (D10)**
  - Summary: D12 runs `window_command` "on every exit path", and a SIGTERM exit is one such path.
    `runproc.End` sends SIGTERM, then SIGKILL after a 5 s grace, and re-lists the owner's descendants three times.
    So a `window_command` started while the child handles SIGTERM can run for up to 30 s, but it is killed within the grace.
    The record does not say which side of D12's rule SIGTERM falls on.
  - Evidence: `internal/runproc/end.go:20-27` (`endGrace`, `endPasses`); `internal/desktophook/window.go:35` (`commandTimeout` of 30 s).
  - Fix: in D12, state that a SIGTERM exit runs no `window_command`, or that it runs one within the grace and is killed with the child.

**Tool calls**
1. Grepped `open_window_command`'s default and `proc.Detach`: I needed the Open-Window row's command line and the detach mode, which the manifest lacked.
2. Grepped `shedadapters/command.go` for how it runs commands: I needed to know whether a detached child could hold the row open.
3. Grepped `command.go`'s functions and timeout lines: the first grep matched nothing.
4. Grepped for the production `Run` seam: I needed to find the row's real runner.
5. Read `battencli/land.go`'s `runRowCommand` and `commandEnv`: I needed the row's output handling and environment, for D10's bound and runproc ownership.
6. Read `dobbyengine/config.go`'s per-task key checks and the board's key list: I needed D7's load-time refusal path.
