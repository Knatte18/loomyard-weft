# Card 3 — window-command-cancel

**Create:**
- `plan:internal/desktophook#AfterPlacement` -> `func AfterPlacement(ctx context.Context, dirs configengine.Dirs, dir string) error`
- `plan:internal/desktophook#AfterPlacementWithTiming` -> `func AfterPlacementWithTiming(ctx context.Context, dirs configengine.Dirs, dir string, commandTimeout, commandWaitDelay, groupKillAfter time.Duration) error`
- `plan:internal/desktophook_test#TestAfterPlacement` -> `func TestAfterPlacement(t *testing.T)`

**Edit:**
- `internal/desktophook#runWindowCommand` -> `func runWindowCommand(ctx context.Context, command, dir string, t timing) error`
- `internal/desktophook#afterWindow`
- `internal/desktophook#timing`
- `internal/desktophook#productionTiming`
- `internal/desktophook#AfterWindowWithTiming`

**Uses:**
- `internal/proc#ConfigureGroupTerminate`
- `internal/desktophook_test#newDirsWithMachineConfig`

**Intent:** Give the window-command runner a way for its caller to cancel a running command and its whole process group (D12), for the placement child.

`AfterPlacement` loads the desktop config like `AfterWindow` and runs `window_command` in `dir` with no `window_wait_ms` sleep,
since the placement child has waited for the window itself.
A config that fails to load returns an error; with the command empty or blank it returns nil at once.
A cancel of `ctx` ends the running command and every process it started, and `AfterPlacement` returns the cancel's error.
`AfterWindow` keeps its behaviour: it waits `window_wait_ms`, then runs the command under `context.Background()`.

`runWindowCommand` takes the caller's `ctx` and bounds it with `t.commandTimeout`.
It starts the command in its own process group through `proc.ConfigureGroupTerminate`,
so a cancel or the timeout sends SIGTERM to the group and SIGKILL after `t.groupKillAfter`, a new `timing` field, 5 seconds in `productionTiming`.
It calls the returned `awaitGone` once the command has returned, and logs each signal the report callback names at Info.
On Windows, where `proc` has no process group, the command alone is killed, as `ConfigureGroupTerminate` documents.
`AfterPlacementWithTiming` in `export_test.go` runs `AfterPlacement` under caller-chosen bounds.
`AfterWindowWithTiming` passes its `commandWaitDelay` as `groupKillAfter`, so `TestAfterWindow`'s timeout row runs under a stated group kill.

`TestAfterPlacement` is `tier1`, since it runs a real shell, in `window_test.go`'s `!windows` file, with rows:
- `UnsetCommandRunsNothing`, which reaches `AfterPlacement`'s own early return;
- `ConfigThatFailsToLoadIsAnError`: `window_wait_ms: -1` returns an error naming the key;
- `SetCommandRunsAtOnce`: `window_wait_ms: 600000` and `pwd > marker`; the call returns within a minute and the marker names the window directory, so no wait ran;
- `CancelEndsTheCommandAndItsGroup`: the command `sleep 300 & echo $! > child; wait`; once `child` exists the test cancels `ctx`;
  the call returns an error, and the pid in `child` is gone (`proc.IsAlive` false) within a minute.
The cut-off at the command timeout is `TestAfterWindow`'s row, through the same `runWindowCommand`.

**ImpactSummary:** The private runner takes a context and a process group; `AfterWindow`'s two callers (`lym reed window`, `lym ide`) see the same behaviour, and the new entry point has no caller until card 11.
