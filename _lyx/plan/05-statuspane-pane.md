# Card 5 — statuspane-pane

**Create:**
- `internal/statuspane/doc.go`
- `internal/statuspane#Frame`
- `internal/statuspane#Pane`
- `internal/statuspane#Pane.Run`
- `internal/statuspane#TerminalSize`
- `internal/statuspane#ReportFromEnv`
- `internal/statuspane#StatusCache`
- `internal/statuspane#NewStatusCache`
- `internal/statuspane#StatusPoll`
- `internal/statuspane#StatusCache.Poll`
- `internal/statuspane#FormatMinutes`
- `internal/statuspane#TestFormatMinutes`
- `internal/statuspane/pane_test.go`
- `internal/statuspane/cache_test.go`
- `internal/statuspane/testmain_test.go`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`
- `cmd/lyx/testdata/test-budget.yaml`
- `pattern/PATTERN-daemon-wakeups.md`
- `docs/overview.md`

**Intent:**
Add `internal/statuspane`, the one implementation of the pane mechanics the runs overview (card 9) and the gate checklist (card 7) share: a redraw-in-place pane loop, a status-file reader and the panes' time format.
It takes every path and size it works on as values (`PATTERN-told-geometry`) and imports the standard library, `internal/shedengine`, `internal/agentname`, `internal/logger` and `golang.org/x/term`.

`pane.go`:
- `Frame` holds one poll's `Lines []string`, every content line, and `Fit func(rows int) []string`, which renders those lines into fewer rows, its last row a summary line naming how many are not shown; each renderer owns its own `Fit`.
- `Pane` holds `Out io.Writer`, `Size func() (cols, rows int, err error)`, `Report func(rows int) error` (nil reports nothing) and `Sleep func(time.Duration)`.
- `Run` polls until `ctx` ends.
  Each poll it calls `poll`; when `len(Lines)` differs from the count last reported successfully it calls `Report`, and on an error logs it at Debug and reports again at the next poll; it reads `Size`, uses `Lines` when they fit the rows and `Fit(rows)` otherwise (a failed `Size` keeps every line), and cuts each line to `cols` so none wraps; it redraws only when the text or the size differs from the last redraw.
  A redraw homes the cursor, writes each line followed by an erase-to-end-of-line and a newline except after the last line, then erases below, so the pane is redrawn in place and never scrolls; the cursor is hidden once at the start.
  The wait between polls starts at one second, doubles while a poll changes nothing, up to an eight-second ceiling, and returns to one second on any change, as `PATTERN-daemon-wakeups` requires; both values are named constants.
  Bound: it reads and writes nothing but its told seams and `Out`.
- `TerminalSize(f)` reads the columns and rows of the terminal on `f` through `term.GetSize`, so the pane reads its own height and a summary line is right before reed has applied a reported count.
- `ReportFromEnv(getenv, set)` returns a report func that calls `set` with the `agentname.StrandNameEnv` and `TMUX_PANE` values `getenv` gives (production passes `os.Getenv`), or nil when either is empty, so a hand-run pane outside a strand reports nothing.

`cache.go`: `StatusCache.Poll(path)` reads a `shedengine.Status` without the run's status lock, which is safe because `internal/state` writes it atomically, so it never blocks or slows a run.
It stats the file and decodes it only when its modification time differs from the last poll's.
`StatusPoll` reports `Status`, `Found` and `Unreadable`.
An absent file is `Found` false and no failure.
A failed stat, read or decode is a failure: the first in a row returns the last good status (`Found` false when there was none), and a second in a row, or a later poll of the same failed modification time, returns `Unreadable`; the next good decode clears the count.

`FormatMinutes(d)`, in `pane.go`, renders a duration in whole minutes as `12m`, `1h05m` or `2d03h`, the one time format both panes use (shared decision `minute-resolution-times`), so a pane line changes at most once a minute.
`TestFormatMinutes` pins each of its format boundaries.

`unboundedWaitFiles` lists `internal/statuspane/pane.go`, and `PATTERN-daemon-wakeups.md`'s "What counts" list names the status panes' loop.
`docs/overview.md`'s package tree gains the `internal/statuspane/` line.

Tests (all untagged, `t.Parallel`, with a fake clock and a fake size seam):
- `TestPane_RunAndReportFromEnv`, named for both behaviours it pins, table-driven over a scripted poll sequence, with the context cancelled from the fake `Sleep` after the last poll: an unchanged frame redraws once, and the waits double from one second to the ceiling; a changed frame redraws and the wait returns to one second; a frame taller than the told rows prints `Fit`'s rows and never more; a size change alone redraws; a line wider than the told columns is cut; a `Report` that fails on the first poll is called again with the same count on the second and not on the third; a report func nil reports nothing.
  A sub-test pins `ReportFromEnv` over a map-backed `getenv`: with both variables set the returned func calls `set` with their values and the count, and with either empty the result is nil.
- `TestFormatMinutes`, in `pane_test.go`, table-driven with exact-string rows: a sub-minute duration (`0m`), 12m, 59m, 60m (`1h00m`), `1h05m`, `23h59m`, 24h (`1d00h`) and `2d03h`, so each format boundary is pinned.
- `TestStatusCache_Poll` over a temp file: an absent file is not found; a written status is found; an unchanged modification time is not re-decoded, pinned by overwriting the content while restoring the time; one undecodable write keeps the previous status, a second poll of it reports unreadable, and a good write clears it.
- `testmain_test.go` runs the package through `tmuxkit.Main` (`PATTERN-test-isolation`).
- `test-budget.yaml` gains `internal/statuspane: 3`.

**ImpactSummary:** a new package and doc and scan-list lines; nothing calls it until cards 7 and 9.
