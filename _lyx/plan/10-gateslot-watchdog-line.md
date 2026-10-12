# Card 10 — gateslot renders and finds the CPU watchdog's quit line

**Create:**
- `internal/gateslot/watchdogline.go`
- `internal/gateslot/watchdogline_test.go`
- `internal/gateslot#WatchdogLine`
- `internal/gateslot#FindWatchdogQuit`
- `internal/gateslot#BusyQuitWayForward`
- `internal/gateslot#TestWatchdogLine`

**Edit:**
- `internal/gateslot/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D7's shared seam: the line `lyx gate test`'s CPU watchdog prints after a quit is rendered and found in one place.

`internal/gateslot/watchdogline.go` holds the line every verify gate reads (cards 12 to 14), beside the config key it names:
- `WatchdogLine` renders `lyx gate test: CPU watchdog: <binary> (pid <pid>) kept a CPU core busy for <window>; sent SIGQUIT, so its goroutine stacks follow in the output`, the window in `time.Duration`'s own form.
- `FindWatchdogQuit` finds the first such line anywhere in a log and returns its window text.
- `BusyQuitWayForward` is the shared way forward a halt names after a watchdog quit.

`gateslot`'s package doc gains a "Watchdog line" section: the line the gate test's CPU watchdog prints after a quit is rendered by `WatchdogLine` and found by `FindWatchdogQuit`, and `BusyQuitWayForward` is the way forward after one.

`TestWatchdogLine`, untagged, a table whose rows each give a log and the wanted window and `ok`:
- a log that is exactly `WatchdogLine("/tmp/x/a.test", 42, 2*time.Minute)` finds `2m0s`;
- that line amid other test output, after it a `SIGQUIT: quit` header as a real quit's log carries, finds `2m0s`;
- a log of test output with no watchdog line finds nothing.
`internal/gateslot` has 11 top-level tests besides `TestMain` against a row of 11, so its budget row rises to 12.

**ImpactSummary:** one new file of exported helpers in `internal/gateslot` with no caller yet; card 11's watchdog writes the line and cards 12 to 14 read it.

**Commit:** 10: gateslot renders and finds the CPU watchdog's quit line
