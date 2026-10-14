# Card 3 — Status pane: full clear on every redraw, and a resize wakes the loop

**Create:**
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseHistory` -> `const eraseHistory = "\x1b[3J"`
- `plan:internal/statuspane#ResizeSignal` -> `func ResizeSignal(ctx context.Context) <-chan struct{}`
- `plan:internal/statuspane#TestResizeSignal_DeliversSIGWINCH` -> `func TestResizeSignal_DeliversSIGWINCH(t *testing.T)`

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane#paneStep`
- `internal/statuspane#rendered`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/statuspane/doc.go`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D3; the fix lives in `statuspane`, so the loom checklist and batten's overview pane both get it.

Redraw: `Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the lines joined by `\n`, with no per-line erase and no erase below.
`eraseToLineEnd` and `eraseBelowCursor` are deleted; their only users are `redraw` and the test helper `rendered`, both on this card.
`rendered` builds the same prefix, so every row's `wantLast` pins the new bytes.

Resize wake: `Pane` replaces its `Sleep func(time.Duration)` field with `After func(time.Duration) <-chan time.Time` (nil means `time.After`) and gains `Resized <-chan struct{}` (nil never fires).
`Run`'s wait between polls becomes a `select` over `ctx.Done()`, `After(wait)` and `Resized`; a receive on `Resized` ends the wait at once and the loop polls again, so the size change it reads redraws and the wait returns to one second.
The back-off itself is unchanged: `initialWait` doubling to `maxWait` while nothing changes, as `PATTERN-daemon-wakeups` requires.
`ResizeSignal(ctx)` returns the channel the two callers pass as `Resized`.
It is declared in two per-OS files: `internal/statuspane/resize_unix.go` (`//go:build !windows`) registers `syscall.SIGWINCH` with `os/signal.Notify`, forwards each signal into a one-slot buffered channel without blocking, so a burst of resizes coalesces into one wake, and stops the notification when `ctx` ends;
`internal/statuspane/resize_windows.go` returns nil, so on Windows the loop keeps its poll.
Bound: the new wake source only shortens a wait; it adds no poll beyond one per delivered resize burst, and `statuspane` still imports only the standard library, `shedengine`, `agentname`, `logger` and `x/term`.

Callers: `checklistCmd` (`internal/loomcli/checklist.go`) and `overviewCmd` (`internal/battencli/overview.go`) drop `Sleep: time.Sleep` and set `Resized: statuspane.ResizeSignal(cmd.Context())` (the overview's `ctx`), leaving `After` nil.
Drop any import those files then no longer use.

Tests in `TestPane_RunAndReportFromEnv` (`internal/statuspane/pane_test.go`), whose file header comment then names the fake size, `After` and `Resized` seams in place of the sleep seam:
- The harness's `Sleep` seam becomes `After`: it records `d` into `waits`, cancels the context after the last step as today, and returns a channel already holding a value.
- `paneStep` gains `resize bool`: for a step with `resize` set, the `After` call before it returns a channel that never fires and the harness sends one value on the pane's buffered `Resized` channel instead, so only the resize can end that wait; the harness runs `Run` in a goroutine and fails the row with `t.Fatal` when `Run` has not returned within five seconds of real time, which is how a loop that ignores `Resized` fails instead of hanging the package.
- New row `ResizeWakesTheWaitAndRedrawsAtOnce`: three steps of the same frame, the third with `resize` set and a changed `rows`; want two redraws and waits `1s, 2s, 1s` (the third wait resets because the resize redrew).
- The existing row `ChangedFrameRedrawsAndTheWaitReturnsToOneSecond` changes its frames to two steps of three long lines, then one step of one short line, keeping its waits `1s, 2s, 1s` and two redraws; its `wantLast` is exactly `rendered("x")`, which begins with cursor home, erase screen and erase history and holds no byte of the old lines, so a shorter, narrower frame over a longer, wider one is pinned without a new row.
- Every existing row's `wantLast` keeps its lines and gets the new prefix through `rendered`.

`TestResizeSignal_DeliversSIGWINCH` is a new Tier-1 test in a new file `internal/statuspane/resize_unix_test.go` (`//go:build !windows`, package `statuspane`): it calls `ResizeSignal(ctx)`, sends `syscall.SIGWINCH` to its own process with `syscall.Kill(os.Getpid(), syscall.SIGWINCH)` (ignored by default, so harmless), and fails unless the channel delivers within five seconds of real time.
It asserts nothing about coalescing, whose timing depends on asynchronous signal delivery.
It does not run in parallel, since signal delivery is process-global.
`cmd/lyx/testdata/test-budget.yaml`: raise `internal/statuspane` from 6 to 7.

`internal/statuspane/doc.go`: the redraw paragraph says a redraw homes the cursor, erases the screen and the scrollback history and writes the frame, so nothing of an earlier frame stays on screen or in the history tmux pulls back onto a growing pane; it adds that `Resized` ends a wait at once, fed by `ResizeSignal` on platforms with a resize signal, and that the import list is unchanged.

**ImpactSummary:** One package's pane loop and its two callers (loomcli checklist, battencli overview); one field renamed on an internal-only struct, no other users; the loop's tests stay in the existing table, and one new test pins the unix resize source.

**Commit:** 3: statuspane: clear screen and history on every redraw, wake on resize
