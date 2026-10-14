# Card 3 — status-pane-clear

**Create:**
- `internal/statuspane/resize_unix.go`
- `internal/statuspane/resize_windows.go`
- `plan:internal/statuspane#ResizeWake` -> `func ResizeWake(ctx context.Context) <-chan struct{}`
- `plan:internal/statuspane#eraseScreen` -> `const eraseScreen = "\x1b[2J"`
- `plan:internal/statuspane#eraseHistory` -> `const eraseHistory = "\x1b[3J"`
- `internal/statuspane/resize_unix_test.go`
- `plan:internal/statuspane#TestResizeWake_ForwardsSIGWINCH` -> `func TestResizeWake_ForwardsSIGWINCH(t *testing.T)`

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

**Intent:**
Decision D3: every status pane redraw clears the screen and the scrollback history, and a terminal resize wakes the pane loop at once.
The fix lives in `statuspane`, so the loom checklist and batten's overview pane both get it.

Redraw: `Pane.redraw` writes `cursorHome`, then `eraseScreen` (`ESC [ 2 J`), then `eraseHistory` (`ESC [ 3 J`), then the frame's lines joined by newlines, with no newline after the last.
The per-line `eraseToLineEnd` and the trailing `eraseBelowCursor` go, since the full clear leaves nothing of an earlier frame on screen or in the history tmux pulls back onto a growing pane.

Resize wake: `Pane`'s `Sleep func(time.Duration)` field is replaced by two fields.
`After func(time.Duration) <-chan time.Time` returns the channel the wait between polls ends on; nil means `time.After`.
`Wake <-chan struct{}` is a channel whose receive ends the wait at once; nil never wakes.
`Pane.Run` waits between polls with a `select` on `ctx.Done()`, `After(wait)` and `Wake`.
A wake polls at once, and the poll redraws when the size or text changed, as any poll does; the next wait after a redraw is the one-second floor as today, so the idle back-off required by `PATTERN-daemon-wakeups` stays.
Bound: a wake only shortens one wait, it never adds a periodic wake, and only the resize signal feeds it.

`ResizeWake` is the production wake source.
In `resize_unix.go` (`//go:build !windows`) it subscribes to `syscall.SIGWINCH` through `os/signal`, forwards each signal as a non-blocking send on a one-slot channel, and stops the subscription when ctx ends.
In `resize_windows.go` it returns nil, so the loop keeps its poll on a platform with no resize signal.
Both import only the standard library, which keeps `statuspane` inside its stated import set.

Callers: `loomCLI.checklistCmd` and `battenCLI.overviewCmd` drop `Sleep: time.Sleep` and set `Wake: statuspane.ResizeWake(<the command's context>)`, leaving `After` nil.

Tests, as rows of `TestPane_RunAndReportFromEnv` except the one `ResizeWake` test below:
- `rendered` builds the new bytes (home, erase screen, erase history, lines joined by newlines), so every existing row's `wantLast` pins that each redraw begins with cursor home, erase screen and erase history.
- The harness's `Sleep` seam becomes an `After` fake that records each wait and returns a closed channel, cancelling the context after the last step as today, so the existing `wantWaits` rows keep their meaning.
- The existing row "ChangedFrameRedrawsAndTheWaitReturnsToOneSecond" is reshaped to shrink rather than grow: two polls of a three-line frame with lines wider than the later one, then a one-line narrow frame, with `wantReports` `{3, 1}`; its `wantLast` is exactly `rendered` of the short frame, which pins that a shorter, narrower frame drawn over a longer, wider one leaves no line of the old.
  The row is renamed "ShorterNarrowerFrameRedrawsOverTheOldAndTheWaitReturnsToOneSecond"; no new row is added for this.
- New row "ResizeWakeRedrawsWithoutWaitingOutTheBackoff": `paneStep` gains a `wake bool` field; for a step that sets it the harness sends on `Wake` and the `After` fake returns a `time.After` channel of ten seconds of real time, marking a flag when that channel is the one the loop received from, so the loop is expected to proceed only through the wake; the step after it reports a new size, and the row asserts the redraw count, that the wait after the redraw is one second, and that the timeout flag is unset.
  The row fails, after at most ten seconds and never by hanging, if the loop ignores `Wake`.
- `TestResizeWake_ForwardsSIGWINCH`, a new top-level test in the new file `internal/statuspane/resize_unix_test.go` (`//go:build !windows`, since `syscall.SIGWINCH` exists only there), calls `ResizeWake(ctx)`, sends the process `syscall.SIGWINCH` through `os.FindProcess(os.Getpid())`, and asserts a receive on the returned channel within a bounded wait.
  The signal is process-global but `SIGWINCH`'s default action is to ignore it, so the test stays parallel-safe.
  `cmd/lyx/testdata/test-budget.yaml` raises `internal/statuspane` from 6 to 7.

**ImpactSummary:** `statuspane`'s exported `Pane` loses `Sleep` and gains `After` and `Wake`; its two callers, `loomcli`'s checklist and `battencli`'s overview, change one field each; the redraw byte sequence changes for both panes.

**Verify:** GOOS=windows go vet ./internal/statuspane
