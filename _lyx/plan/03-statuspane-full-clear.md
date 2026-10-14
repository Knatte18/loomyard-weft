# Card 3 — statuspane-full-clear

**Create:**
- `internal/statuspane/resize_other.go`
- `internal/statuspane/resize_windows.go`
- `plan:internal/statuspane#WatchResize` -> `func WatchResize() (<-chan os.Signal, func())`
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseScrollback` -> `const eraseScrollback = "\x1b[3J"`
- `internal/statuspane/resize_other_test.go`
- `plan:internal/statuspane#TestWatchResize_DeliversSIGWINCH` -> `func TestWatchResize_DeliversSIGWINCH(t *testing.T)`

**Delete:**
- `internal/statuspane#eraseToLineEnd`
- `internal/statuspane#eraseBelowCursor`

**Edit:**
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#Pane.redraw`
- `internal/statuspane#rendered`
- `internal/statuspane#paneStep`
- `internal/statuspane#TestPane_RunAndReportFromEnv`
- `internal/statuspane/doc.go`
- `internal/loomcli#loomCLI.checklistCmd`
- `internal/battencli#battenCLI.overviewCmd`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Make every status pane redraw a full clear and wake the pane loop on a resize (D3), in `statuspane`, so the loom checklist and batten's overview both get it.

Redraw: `Pane.redraw` writes `cursorHome`, `eraseScreen` (`ESC [ 2 J`) and `eraseScrollback` (`ESC [ 3 J`), then the lines joined by newlines with no per-line erase and no erase below.
The per-line and below-cursor erases are deleted with their constants.
A full clear leaves nothing of an earlier, longer or wider frame on screen or in the history tmux pulls back onto a growing pane.

Resize wake: `Pane` drops its `Sleep func(time.Duration)` field and gains `After func(time.Duration) <-chan time.Time` (nil means `time.After`), `Now func() time.Time` (nil means `time.Now`) and `Resized <-chan os.Signal` (nil never fires).
`Pane.Run` waits between polls on a `select` over `ctx.Done()`, `After(wait)` and `Resized`.
A resize ends the wait without waiting out the back-off, but never polls sooner than `initialWait` after the previous poll (`PATTERN-daemon-wakeups`): when the previous poll is at least `initialWait` old by `Now`, the loop polls at once; otherwise it waits on `After` for the rest of that second, so a burst of resize signals during a window drag costs at most one poll a second.
The back-off itself is unchanged: one second after a change, doubling to eight while nothing changes.
`WatchResize` returns the channel and a stop func: in `resize_other.go` (`//go:build !windows`) it is `signal.Notify` on `syscall.SIGWINCH` into a one-slot buffered channel, and stop is `signal.Stop`; in `resize_windows.go` it returns a nil channel and a no-op stop, since Windows has no resize signal and the loop keeps its poll there.
Bound: the signal only shortens a back-off wait, never below one second between polls, and never changes what a redraw writes.

Callers: `checklistCmd` and `overviewCmd` replace `Sleep: time.Sleep` with `Resized` from `statuspane.WatchResize()`, deferring its stop, and leave `After` and `Now` nil.
That wiring is a one-line composition inside each verb's terminal loop and gets no test of its own; the source and the loop are tested below.

Tests: `rendered` builds the new redraw bytes (`cursorHome`, `eraseScreen`, `eraseScrollback`, lines joined by newlines), and every row of `TestPane_RunAndReportFromEnv` compares its last redraw to `rendered(...)` exactly.
So `ChangedFrameRedrawsAndTheWaitReturnsToOneSecond`, `FrameTallerThanTheTermRowsPrintsFitRowsAndNeverMore` and `LineWiderThanTheColumnsIsCut` carry the criterion that a shorter, narrower frame drawn over a longer, wider one leaves no line of the old: their redraw must begin with the full clear and hold no erase-to-line-end, and the redraw bytes do not depend on the previous frame, so no further row adds a branch.
The table's seam moves from `Sleep` to `After` and `Now`: the test keeps a fake clock; its `After` records the wait, advances the clock by it, cancels the context after the last step as `Sleep` did, and returns a channel that already holds a value; its `Now` reads the clock.
`paneStep` gains `resize bool` and `elapsed time.Duration`: for a step with `resize` set, the test's `After` advances the clock by `elapsed` only, sends on a buffered `Resized` channel it gave the pane, and returns a channel that never fires, so only the resize can end that wait.
Rows that use a resize run under a `context.WithTimeout` of a few seconds and assert the context did not time out, so an implementation that ignores `Resized` fails on that assertion instead of hanging the binary.
New rows:
- `ResizeWakesThePaneWithoutWaitingOutTheBackOff`: two unchanged steps at 80x24, the second marked `resize` with `elapsed` 1.5s inside its 2s wait, then a step at 100x30; the run redraws twice, records the waits `1s, 2s` and no third wait before the last poll, which an implementation that waits only on `After` cannot do.
- `ResizeWithinASecondOfThePollWaitsTheRestOfThatSecond`: the same with `elapsed` 300ms; the waits recorded are `1s, 2s, 700ms`, then the redraw at the new size.
`TestWatchResize_DeliversSIGWINCH` (`resize_other_test.go`, `//go:build !windows`) calls `WatchResize`, sends `syscall.SIGWINCH` to its own process, and asserts a value arrives on the channel within a bounded wait, then calls stop; it is not parallel, since the signal is process-global, and Go ignores an unhandled SIGWINCH, so no other test is disturbed.
`test-budget.yaml`'s `internal/statuspane` row rises by one.
`doc.go` replaces the redraw sentence with the full-clear one, names the resize wake, its one-second floor and its Windows fallback, and adds `os/signal` and `syscall` to the imports it lists as standard library.

**ImpactSummary:** Pane's seam changes from Sleep to After, Now and Resized; its two callers (loomcli checklist, battencli overview) update in this card; redraw bytes change for both panes.

**Verify:** GOOS=windows CGO_ENABLED=0 go vet ./internal/statuspane
