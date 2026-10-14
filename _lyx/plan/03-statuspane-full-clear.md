# Card 3 — statuspane-full-clear

**Create:**
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseHistory` -> `const eraseHistory = "\x1b[3J"`
- `plan:internal/statuspane#sleepOrWake` -> `func sleepOrWake(d time.Duration, wake <-chan struct{})`
- `plan:internal/statuspane#ResizeSignal` -> `func ResizeSignal() (resized <-chan struct{}, stop func())`
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `plan:internal/statuspane#checkResizeSignal` -> `func checkResizeSignal(t *testing.T)`
- `internal/statuspane/resize_unix_test.go`
- `internal/statuspane/resize_windows_test.go`

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

**Intent:** D3: every status pane redraw clears screen and history, and a terminal resize wakes the pane loop at once; the loom checklist and batten's overview pane both get it through `statuspane`.

`Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the lines joined by `"\n"` with no newline after the last, in place of the per-line erase-to-end and the trailing erase-below.
Delete the two constants it no longer uses; no other file names them but `pane_test.go`'s `rendered`.

`Pane` gains a field `Resized <-chan struct{}`: a receive on it means the terminal was resized; nil means no resize signal, and the loop keeps its poll.
`Pane.Sleep` changes type from `func(time.Duration)` to `func(d time.Duration, wake <-chan struct{})`: it waits d, or less when wake delivers.
`Run` passes `p.Resized` as wake on every wait, and a nil `Sleep` means `sleepOrWake`, which returns after d or at the first receive from wake, whichever is first (a nil wake never delivers).
So a resize ends the wait at once, the next poll reads the new size and redraws, and the wait returns to one second; the idle back-off is unchanged while nothing changes, per `PATTERN-daemon-wakeups`, since only a resize shortens a wait.

`ResizeSignal` returns a channel fed by the platform's resize signal and a stop func that ends the feed.
In `resize_unix.go` (`//go:build !windows`) it subscribes to `syscall.SIGWINCH` with `os/signal.Notify` on a buffered channel and forwards each signal as a non-blocking send on a buffered `chan struct{}` (capacity 1, so bursts coalesce) from one goroutine, which `stop` ends after `signal.Stop`.
In `resize_windows.go` it returns a nil channel and a no-op stop, so the loop keeps its poll there.
Both files declare the one `ResizeSignal`; the imports stay standard library.

`checklistCmd` and `overviewCmd` build their `Pane` with `Resized` from `statuspane.ResizeSignal()`, defer its stop, and drop their `Sleep: time.Sleep` (the nil default now waits through `sleepOrWake`).

`internal/statuspane/doc.go` replaces the redraw sentence (home, per-line erase, erase below) with home, erase screen and erase history before the frame, adds that a resize ends the wait at once where the platform has a resize signal, and adds `os/signal` and `syscall` to the import list it states.

Tests, all in `TestPane_RunAndReportFromEnv`:
- `rendered` builds `cursorHome + eraseScreen + eraseHistory` followed by the lines joined by `"\n"`, so every existing row's `wantLast` asserts the new prefix; the redraw count keeps counting `cursorHome`.
- The table's `Sleep` fake takes the new signature, records d as before, and records whether the wake it was handed is the pane's `Resized` channel; the loop builds the pane with a non-nil `Resized` channel and every row asserts the fake always got it.
- New row `ShorterNarrowerFrameRedrawsFromAClearedScreen`: a frame `("aaaa","bbbb","cccc")` at 80x24, then `("a")` at 40x24; two redraws, and the last redraw equals `rendered("a")` exactly, so nothing of the old frame survives it.
- New subtest `NilSleepWakesOnAResize`: a `Pane` with nil `Sleep`, a `Resized` channel buffered with two values, and a poll that cancels the context on its second call; it asserts two polls happened and the channel is drained.
  `sleepOrWake` ignoring wake would wait the one-second wait and leave both values queued, so the drain assertion fails; with wake honoured no wall-clock wait elapses.
- New subtest `ResizeSignalForwardsSIGWINCH`, skipped on Windows (where `ResizeSignal` returns a nil channel, which the subtest asserts instead) and not parallel, since it signals its own process, process-global state it names: it calls `ResizeSignal`, sends `syscall.SIGWINCH` to `os.Getpid()` and asserts one value arrives within a bounded wait; then it calls `stop`, sends the signal again and asserts no value arrives within a short bounded wait.
  A `ResizeSignal` that never forwards, or keeps forwarding after `stop`, fails it.
  It sits in `resize_unix_test.go` behind the same `//go:build !windows` constraint, beside the Windows assertion in `resize_windows_test.go`, both subtests of no new top-level test: each file declares `checkResizeSignal`, which the `TestPane_RunAndReportFromEnv` subtest calls.

**ImpactSummary:** `statuspane`'s redraw bytes and `Pane.Sleep`'s type change; the two pane builders (loom checklist, batten overview) are its only users and change with it, and the only test asserting the bytes is `pane_test.go`.

**Commit:** 3: Clear screen and history on every status pane redraw and wake on resize
