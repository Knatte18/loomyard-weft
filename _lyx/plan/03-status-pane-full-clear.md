# Card 3 — The status pane clears screen and history on every redraw, and a resize wakes it

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane#rendered`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/statuspane/doc.go`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Create:**
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseScrollback` -> `const eraseScrollback = "\x1b[3J"`
- `plan:internal/statuspane#sleepOrWake` -> `func sleepOrWake(d time.Duration, wake <-chan struct{})`
- `plan:internal/statuspane#ResizeSignal` -> `func ResizeSignal(ctx context.Context) <-chan struct{}`
- `plan:internal/statuspane#raiseResize` -> `func raiseResize(t *testing.T)`
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `internal/statuspane/resize_unix_test.go`
- `internal/statuspane/resize_windows_test.go`

**Intent:**
Decision D3, in `statuspane`, so the loom checklist and batten's overview pane both get it.

Redraw: `Pane.redraw` writes `cursorHome`, `eraseScreen` (`ESC [2J`) and `eraseScrollback` (`ESC [3J`), then the lines joined by `"\n"` with nothing after the last.
This replaces the per-line `eraseToLineEnd` and the trailing `eraseBelowCursor`, which this card deletes along with their only other user, the test helper `rendered`.
`rendered` changes to build the same new byte shape.
So a shorter, narrower frame drawn over a longer, wider one leaves nothing of the old on screen or in the history tmux pulls back onto a growing pane.

Wake on resize:
- `Pane` gains a field `Resized <-chan struct{}`, which receives once per terminal resize; a nil channel never receives.
- `Pane.Sleep`'s type becomes `func(d time.Duration, wake <-chan struct{})`: it waits `d` or until `wake` receives, whichever comes first.
  `Run` passes `p.Resized` as `wake` on every wait.
  A nil `Sleep` uses `sleepOrWake`, a `select` over a `time.NewTimer(d)` (stopped on return) and `wake`.
- The poll after a woken wait reads the new size and redraws, as a size change does today.
  The back-off schedule is unchanged, so `PATTERN-daemon-wakeups` holds: an idle pane still waits up to eight seconds, and only a resize cuts a wait short.
- `ResizeSignal(ctx)` returns the channel a caller sets as `Resized`.
  `resize_unix.go` (`//go:build !windows`) subscribes `signal.Notify` to `syscall.SIGWINCH`, forwards each signal as a non-blocking send on a one-slot buffered channel, and calls `signal.Stop` when `ctx` ends.
  `resize_windows.go` returns nil, because Windows has no resize signal, so the loop keeps its poll there.
- `checklistCmd` and `overviewCmd` drop `Sleep: time.Sleep` and set `Resized: statuspane.ResizeSignal(<the command's context>)`, so the default sleep is used.
- `doc.go` replaces the redraw sentence ("A redraw homes the cursor, writes each line followed by an erase …") with the full clear.
  It also adds the resize wake and the imports it now takes from the standard library (`os/signal`, `syscall`).
- Open risk accepted by the record: clearing history on every redraw discards an operator's copy-mode position in a status pane.

Tests, in `TestPane_RunAndReportFromEnv`:
- The table's fake `Sleep` takes `(d, wake)`, records `d` as before, and asserts every call received the pane's `Resized`.
  Each row sets `Resized` to one shared, never-sent channel.
- New row "ShorterNarrowerFrameOverALongerWiderOneLeavesNothingOfTheOld": step 1 `frameOf("a long first line", "second", "third")`, step 2 `frameOf("a")`, both at 80×24.
  It expects `wantRedraws: 2`, `wantWaits: [1s, 1s]`, `wantReports: [3, 1]` and `wantLast: rendered("a")`, the criterion's shorter, narrower shape; the table's exact `wantLast` comparison already pins the leading full clear.
- New subtest "SleepOrWakeReturnsOnAWake": `sleepOrWake(time.Hour, ch)` with one value already in a buffered `ch` returns at once.
  A broken wake hangs the test until `go test`'s timeout fails it.
- New subtest "ResizeSignalWakesOnAResize": not parallel, because a signal is process-global.
  It calls `ResizeSignal(ctx)`, then `raiseResize(t)`, then receives from the channel.
  `raiseResize` lives in `resize_unix_test.go` (`syscall.Kill(os.Getpid(), syscall.SIGWINCH)`, whose default action is to ignore it) and in `resize_windows_test.go` (`t.Skip` naming the missing resize signal), so the subtest compiles on every OS and adds no top-level test.

**ImpactSummary:** Every status pane redraw now clears the screen and scrollback instead of erasing per line; `Pane.Sleep` changes shape, its two production users (loom checklist, batten overview) switch to the default sleep with a resize channel, and the one pane test table moves to the new byte shape.

**Commit:** 3: statuspane: clear screen and history on every redraw, wake on resize
