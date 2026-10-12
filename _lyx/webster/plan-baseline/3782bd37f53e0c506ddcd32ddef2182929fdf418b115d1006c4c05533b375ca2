# Card 3 — network-gate

**Create:**
- `internal/pauserecord#Waiter`
- `internal/pauserecord#WaitUncovered`
- `internal/pauserecord#NetOp`
- `internal/pauserecord#NetDir`
- `internal/pauserecord#ListNetOps`
- `internal/pauserecord#Gate`
- `internal/pauserecord#InstallGate`
- `internal/pauserecord#SetGateSlug`
- `internal/pauserecord#Network`
- `internal/pauserecord/hold_test.go`

**Edit:**
- `internal/pauserecord/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the wait every held entry point shares and the process gate that holds network calls (D4), in `hold.go`.

`WaitUncovered` returns once no live entry covers `slug` (`Record.Covering`), and at once when none does.
`Waiter` carries `Notify func(line string)` (nil writes the line to stderr), `Sleep func(ctx context.Context, d time.Duration) error` (nil waits on a timer) and `SlugEntryExempt func() bool`: while it reports true only a hub entry holds the wait, which is D4's exception for a caller whose run holds a parent lock.
The wait polls from one second, doubling to a five-second ceiling while the covering entry is unchanged and returning to one second when it changes (`PATTERN-daemon-wakeups`); it has no deadline and returns the context's error when `ctx` ends.
When the wait starts it calls `Notify` once with `lyx: held by <a hub-wide pause | the pause of <slug>> since <paused_at>; it continues after "lyx resume"` (plus `or "lyx resume <slug>"` for a slug entry), and never again during that wait.
An unreadable record holds nothing: the wait returns at once after one `Notify` line naming the record path and the read error, so a corrupt file never stalls every network call.

The network gate is the process-wide hold network calls go through (cards 5 and 6), installed once by the CLI root (card 23), the way `gitexec.SetKillReporter` is.
`Gate` holds `Geometry` (record path and the process's pair slug), `Strand` (the process's strand name, empty outside a strand), `PID` and `StartTime` (the process's own), `HoldsParentLock func(slug string) bool`, `Notify` and `Sleep`.
`InstallGate` stores it behind an `atomic.Pointer` and returns a restore function that reinstates the previous gate, for tests.
`SetGateSlug(slug)` replaces the installed gate's slug, for a process that runs in the prime on behalf of one pair, and is a no-op when no gate is installed.
`Network(op)` with no gate installed returns a no-op at once.
Otherwise it runs `WaitUncovered` for the gate's current slug with `SlugEntryExempt` set from `HoldsParentLock` of that slug, then writes a `NetOp` marker (`PID`, `StartTime`, `Slug`, `Strand`, `Op`, `UnderParentLock`, `StartedAt`) as `<NetDir>/<pid>-<n>.json`, and returns the function that removes it.
`NetDir(recordPath)` is `net/` beside the record.
`ListNetOps` lists the markers, removing and skipping each one whose process `alive` reports dead, so the pause grace (card 22) and the orch watcher (card 21) see only operations in flight.

Bound: the gate only waits at entry and writes a marker; it fails no call, holds nothing when no gate is installed or the record is unreadable, and its worst case is a call waiting until `lyx resume`.

Tests, in `hold_test.go` (not parallel: the gate is process-wide state): one table test over `WaitUncovered` driving `Sleep` as the clock: no entry returns at once with no line; a hub entry holds until the test resumes it inside `Sleep`, with exactly one `Notify` line naming `lyx resume`; a slug entry holds its slug and not another; `SlugEntryExempt` true passes a slug entry and is held by a hub entry; the wake delays double from one second to the ceiling; an undecodable record returns at once with one line; a cancelled context returns its error.
One test over `Network` and `ListNetOps`: no gate is a no-op writing no marker; an installed gate under a live entry waits, then writes a marker that `ListNetOps` lists and `done` removes; a marker whose process `alive` reports dead is removed and not listed; a gate installed with an empty slug and then given a slug through `SetGateSlug` is held by a live entry for that slug.
`internal/pauserecord/hold.go` joins `unboundedWaitFiles` and the background file's list of unbounded waits; `doc.go` gains the hold and gate paragraphs.

**ImpactSummary:** Adds hold.go to the new leaf package and lists it as an unbounded wait; no caller until cards 5, 6 and 8.
