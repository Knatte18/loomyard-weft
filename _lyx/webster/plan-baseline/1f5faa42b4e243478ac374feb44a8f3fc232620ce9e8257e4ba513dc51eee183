# Card 3 — The status pane clears screen and history on every redraw, and a resize wakes it

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane/doc.go`
- `internal/statuspane#paneStep`
- `internal/statuspane#rendered`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`

**Create:**
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `internal/statuspane#NotifyResize`
- `internal/statuspane#eraseScreen`
- `internal/statuspane#eraseHistory`

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Intent:**
D3: every status pane redraw starts from a cleared screen and a cleared scrollback, and a terminal resize redraws at once, so no line of an earlier, longer or wider frame survives on screen or in the history tmux pulls back onto a growing pane.
The fix lives in `statuspane`, so the loom checklist and batten's overview pane both get it.

`Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the lines joined by a newline with none after the last.
The per-line erase and the erase below are dropped: `eraseToLineEnd` and `eraseBelowCursor` are deleted, and their only users, `Pane.redraw` and the test helper `rendered`, are on this card; `eraseScreen` and `eraseHistory` join the same `const` block.

`Pane` replaces its `Sleep func(time.Duration)` seam with `After func(time.Duration) <-chan time.Time`, nil meaning `time.After`, and gains `Resized <-chan struct{}`, nil meaning no resize signal.
`Pane.Run` waits between polls by selecting on `ctx.Done()`, `After(wait)` and `Resized`, so a resize ends the wait at once without waiting out the back-off, and the next poll redraws for the new size and returns the wait to one second, as a size change does today.
The idle back-off is unchanged, per `PATTERN-daemon-wakeups`: a resize is an external event, never a timer, so the loop still wakes at most once a second while idle.
`NotifyResize` returns a channel that receives one value per terminal resize until `ctx` ends: `resize_unix.go` (`//go:build !windows`) relays `SIGWINCH` through `signal.Notify` from one goroutine that calls `signal.Stop` and returns when `ctx` ends, and `resize_windows.go` returns nil, so the loop keeps its poll there.
Both files import the standard library only, inside the import set `statuspane/doc.go` declares.
`checklistCmd` and `overviewCmd` set `Resized: statuspane.NotifyResize(ctx)` from their command context and drop `Sleep: time.Sleep`.
`statuspane/doc.go`'s redraw sentence says a redraw homes the cursor and erases the screen and the history before the frame, and its wait sentence says a resize ends the wait at once where the platform signals one.

Tests, in `TestPane_RunAndReportFromEnv`:
- `rendered` builds the new byte form, so every row's `wantLast` pins that each redraw begins with cursor home, erase screen and erase history, which is what leaves no line of an earlier, longer or wider frame; `ChangedFrameRedrawsAndTheWaitReturnsToOneSecond` and `FrameTallerThanTheTermRowsPrintsFitRowsAndNeverMore` already redraw over a different earlier frame.
- The loop's fake `After` records each wait and returns a channel that already holds a value, cancelling `ctx` after the last step as the fake `Sleep` does today.
- `paneStep` gains `resize bool`: for such a step the fake `After` sends one value onto the test's 1-buffered `Resized` channel and returns a channel that never fires, so only the resize can end that wait and no earlier wait can take the value.
  A new row "ResizeRedrawsWithoutWaitingOutTheBackOff" holds an unchanged frame for three steps, so the wait has backed off, then a resize step whose next poll reads a new size; it asserts the redraw for the new size and that the wait returns to one second.
  The row runs `Pane.Run` under a context with a ten-second timeout, so a loop that ignores `Resized` fails the row on its redraw count instead of hanging the test binary.

**ImpactSummary:** `Pane`'s seam change reaches its two production users, `loomcli`'s checklist and `battencli`'s overview, both on this card, and its one test; the redraw bytes change for both panes and nothing reads them but the terminal.

**Commit:** 3: Clear screen and history on each status pane redraw and wake on resize
