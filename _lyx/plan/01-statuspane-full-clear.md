# Card 1 — statuspane-full-clear

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Create:**
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseHistory` -> `const eraseHistory = "\x1b[3J"`
- `plan:internal/statuspane#ResizeSignal` -> `func ResizeSignal(ctx context.Context) <-chan struct{}`
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `internal/statuspane/resize_unix_test.go`

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane#rendered`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`
- `internal/statuspane/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D3: the shared status pane loop clears screen and history on every redraw, and a terminal resize wakes it at once.
The fix lives in `statuspane`, so the loom checklist and batten's runs overview both get it.

`Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the lines joined by a newline with no newline after the last.
It no longer writes an erase-to-line-end after each line or an erase-below after the frame, so both constants are deleted; their only users are `redraw` and the test helper `rendered`.
A full clear leaves nothing of an earlier, longer or wider frame on screen or in the history tmux pulls back onto a growing pane.

`Pane` drops its `Sleep func(time.Duration)` field for two new fields:
`After func(time.Duration) <-chan time.Time`, the wait seam, where nil waits with `time.After`;
and `Resized <-chan struct{}`, which delivers one value per terminal resize, where nil means the loop only polls.
`Run` waits between polls in one `select` over `After(wait)`, `Resized` and `ctx.Done()`.
A value on `Resized` ends the wait at once; the next poll reads the new size, which redraws and returns the wait to one second exactly as any other change does.
The idle back-off from one second to eight is unchanged, so `PATTERN-daemon-wakeups` holds: the extra wake comes only from a delivered resize signal, an event, never from a shorter timer.

`ResizeSignal(ctx)` returns the channel the two pane commands put in `Resized`.
In `resize_unix.go` (`//go:build !windows`) it registers `syscall.SIGWINCH` with `signal.Notify` on a buffered channel and forwards each signal to a one-slot `struct{}` channel with a non-blocking send, so a burst of resizes coalesces into one pending wake; when ctx ends it calls `signal.Stop` and the forwarding goroutine returns.
In `resize_windows.go` (`//go:build windows`) it returns nil, so Windows keeps the poll, as D3 states for platforms with no resize signal.
Both files import only the standard library, which `statuspane`'s import set already admits.

`loomCLI.checklistCmd` and `battenCLI.overviewCmd` replace `Sleep: time.Sleep` with `Resized: statuspane.ResizeSignal(<the command's context>)`, the same context they pass to `Run`, and leave `After` nil.

`TestPane_RunAndReportFromEnv` swaps its `Sleep` fake for an `After` fake that records the wait, cancels the context on the last scripted poll, and returns a channel already holding a value.
`rendered` builds the new redraw bytes: `cursorHome`, `eraseScreen`, `eraseHistory`, then the lines.
Every row also asserts that the output holds the clear prefix `cursorHome+eraseScreen+eraseHistory` exactly `wantRedraws` times, so every redraw begins with the full clear.
New rows:
- `ShorterNarrowerFrameOverALongerWiderOneLeavesNothingOfTheOld`: a three-line frame of long lines, then a one-line short frame at the same size; two redraws, and the last redraw equals `rendered("a")`, which carries no byte of the earlier lines.
- `AResizeWakesTheWaitWithoutWaitingOutTheBackoff`: three polls of an unchanged frame back the wait off to four seconds, and a new field `resizeAfterPoll` makes the `After` fake, at that poll, put one value on the row's `Resized` channel and return a channel that never fires; the fourth step reads a larger terminal.
  It wants waits `1s, 2s, 4s, 1s`, two redraws, and the fourth poll reached, which only the resize can reach.
  The row's context carries a one-minute deadline as a hang guard, so a loop that ignored `Resized` fails on its poll count instead of hanging; the passing path waits on no clock.
`pane_test.go`'s header comment names the `After` and `Resized` seams in place of the sleep seam.

A new `internal/statuspane/resize_unix_test.go` (`//go:build !windows`, package `statuspane`) holds `TestResizeSignal`, which pins the real signal wiring the `Resized` rows feed by hand.
It does not run in parallel, since it signals its own process.
It sends `syscall.SIGWINCH` to its own process twice and wants exactly one value on the returned channel, which shows the burst coalesced; after cancelling the context and a third signal, no further value arrives.
Each receive and the no-value check are bounded by a one-second select as a hang guard, so only a failing run waits on it.
The Windows nil needs no row: `Run`'s select over a nil channel is the poll-only path every existing row already runs, since those rows leave `Resized` nil.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/statuspane` from 6 to 7 for it.

`internal/statuspane/doc.go` replaces the per-line erase description with the full clear and states the resize wake and its Windows poll fallback.
`pattern/PATTERN-daemon-wakeups.md`'s status-pane bullet adds that the loop also wakes at once on a terminal resize.

**ImpactSummary:** The status pane loop's redraw bytes and wait seam change; its two callers, the checklist and runs-overview commands, swap `Sleep` for `Resized`.
