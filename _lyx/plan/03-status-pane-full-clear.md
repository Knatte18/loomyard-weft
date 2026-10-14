# Card 3 — status-pane-full-clear

**Create:**
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `plan:internal/statuspane#ResizeSignal` -> `func ResizeSignal(ctx context.Context) <-chan struct{}`
- `plan:internal/statuspane#waitOrWake` -> `func waitOrWake(d time.Duration, wake <-chan struct{})`
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseHistory` -> `const eraseHistory = "\x1b[3J"`
- `internal/statuspane/resize_unix_test.go`
- `internal/statuspane/resize_windows_test.go`
- `plan:internal/statuspane#raiseWinch` -> `func raiseWinch(t *testing.T)`

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane#rendered`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/statuspane/doc.go`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`
- `pattern/PATTERN-daemon-wakeups.md`

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Intent:** D3: every status pane redraw clears the screen and its scrollback history before the frame, and a terminal resize wakes the pane loop at once.
The fix lives in `statuspane`, so the loom checklist and batten's runs overview both get it.

Redraw: `Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the lines joined by `"\n"`, with no per-line `eraseToLineEnd` and no trailing `eraseBelowCursor`; delete those two constants, whose only users are `redraw` and the test helper `rendered`.
Nothing of an earlier, longer or wider frame then survives on screen or in the history tmux pulls back onto a growing pane.
Update `redraw`'s doc comment to say so.

Resize wake: two struct-field changes on `Pane`, stated here since a field takes no arrow.
`Sleep`'s type becomes `func(d time.Duration, wake <-chan struct{})`: it waits `d` or until `wake` receives, whichever comes first; a nil `Sleep` waits through `waitOrWake`.
A new field `Resized <-chan struct{}` receives on a terminal resize; nil, the value on a platform with no resize signal, leaves every wait to its timer.
`Pane.Run` passes `p.Resized` to every wait, so a resize ends the current wait at once and the next poll reads the new size and redraws, which also returns the wait to one second, without waiting out the back-off.
The back-off between polls is otherwise unchanged: one second, doubling to eight while a poll changes nothing.
`waitOrWake` selects on a timer of `d` (stopped on return) and on `wake`; a nil `wake` never receives, so it waits the timer alone.
What this lets through: a burst of resize signals wakes the loop once per signal, each wake one poll and at most one redraw; it adds no periodic wake, so the one-second floor of `PATTERN-daemon-wakeups` still holds for the timer.

`ResizeSignal` lives in two new per-OS files, the one `plan:` handle declared once and defined in each.
`resize_unix.go` (`//go:build !windows`) calls `signal.Notify` for `syscall.SIGWINCH` on a buffered channel and forwards each signal, without blocking, to a returned one-slot `struct{}` channel from a goroutine that calls `signal.Stop` and returns when `ctx` ends.
`resize_windows.go` (`//go:build windows`) returns nil: the loop keeps its poll there.
Update `Pane.Run`'s doc comment for the wake.

Callers: `checklistCmd` (`internal/loomcli/checklist.go`) and `overviewCmd` (`internal/battencli/overview.go`) build their `statuspane.Pane` without `Sleep: time.Sleep`, which no longer has the field's type, and set `Resized: statuspane.ResizeSignal(cmd.Context())` (the overview passes its own `ctx`).
Each file keeps its `time` import only where another use remains.

Tests, all in `TestPane_RunAndReportFromEnv` so `internal/statuspane` keeps its budget row of 6:
- `rendered` writes the new redraw bytes: `cursorHome`, `eraseScreen`, `eraseHistory`, then the lines joined by `"\n"`; every existing row's `wantLast` then pins the new redraw exactly.
- The table's fake `Sleep` takes the new signature, records each `d` as today and records each `wake` it receives.
- A new row, "AShorterNarrowerFrameOverALongerWiderOneLeavesNoLineOfTheOld": steps `frameOf("a long first line", "b", "c")` then `frameOf("x")` at the same size; `wantRedraws` 2, `wantWaits` `[1s, 1s]`, `wantReports` `[3, 1]`, `wantLast` `rendered("x")`, so the last redraw holds the clear sequence and no byte of the old frame.
- A new row field `resized bool` sets `pane.Resized` to a channel the row owns, and the run asserts every recorded `wake` is that channel, or nil when the field is false; one new row "TheWaitIsHandedTheResizeChannel" sets it over two unchanged polls.
- A new subtest "ARunWithNoSleepSeamIsWokenByAResize" drives `Pane.Run` itself with a nil `Sleep` and a buffered `Resized` channel that the poll func fills before each return, over forty polls of an unchanged frame, the last of which cancels the context; `Run` runs in a goroutine and the subtest fails if it has not returned within thirty seconds.
  A `Run` that ignored `Resized` on its nil-`Sleep` default would wait out the back-off, one, two, four, then eight seconds per poll, far past that bound, while a woken `Run` returns at once; the bound is wall-clock only on failure.
- A new subtest "WaitOrWakeReturnsOnAWakeAndOtherwiseOnTheTimer": `waitOrWake(time.Hour, wake)` with `wake` already holding a value returns, which a wait ignoring `wake` would not within the test timeout; `waitOrWake(time.Millisecond, nil)` returns once the timer fires.
- A new subtest "ResizeSignalDeliversAWinch" calls `ResizeSignal` with a cancellable context, then `raiseWinch(t)`, and asserts the returned channel receives, failing after ten seconds without one; it cancels the context after.
  `raiseWinch` is a per-OS test helper: `resize_unix_test.go` (`//go:build !windows`) sends the test process `SIGWINCH` with `syscall.Kill(os.Getpid(), syscall.SIGWINCH)`, and `resize_windows_test.go` (`//go:build windows`) calls `t.Skip`, since Windows has no resize signal and `ResizeSignal` returns nil there.
  `SIGWINCH` is ignored by default, so the signal is process-global but harms no parallel test.

Docs: `internal/statuspane/doc.go` replaces the redraw sentence (each line followed by an erase to the end of its row, erasing below) with the full clear, and states that a resize wakes the loop at once on platforms with a resize signal, through `ResizeSignal`, and that the loop polls alone elsewhere.
`pattern/PATTERN-daemon-wakeups.md`'s status-pane bullet adds that the loop also wakes at once on a terminal resize signal.

**ImpactSummary:** statuspane's redraw bytes, `Pane.Sleep`'s type and a new `Pane.Resized` field; the two pane commands in loomcli and battencli rebuild their `Pane` literal; one background file line.

**Commit:** 3: Clear screen and history on every status pane redraw, wake on resize
