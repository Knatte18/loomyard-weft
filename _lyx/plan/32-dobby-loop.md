# Card 32 — dobby-loop

**Create:**
- `internal/dobbyengine/loop.go`
- `plan:internal/dobbyengine#Deps` -> `type Deps struct`
- `plan:internal/dobbyengine#ErrDobbyRunning` -> `var ErrDobbyRunning = errors.New("dobby: another dobby holds this worktree's lock")`
- `plan:internal/dobbyengine#Run` -> `func Run(ctx context.Context, geom Geometry, deps Deps) error`
- `plan:internal/dobbyengine#daemon` -> `type daemon struct`
- `plan:internal/dobbyengine#daemon.tick` -> `func (d *daemon) tick(ctx context.Context, now time.Time) error`
- `plan:internal/dobbyengine#nextWait` -> `func nextWait(current time.Duration, inFlight, event bool) time.Duration`
- `plan:internal/dobbyengine#TestRun_LockOwnerAndWakeups` -> `func TestRun_LockOwnerAndWakeups(t *testing.T)`
- `plan:internal/dobbyengine#TestNextWait` -> `func TestNextWait(t *testing.T)`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`
- `internal/dobbyengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/dobbyengine#Geometry`
- `plan:internal/dobbyengine#State`
- `plan:internal/dobbyengine#UpdateState`

**Intent:**
Build dobby's daemon loop: one per beam worktree, waking cheaply, exiting when it is not its own (D1, D13).
`Deps` holds every seam the loop reads, told by the hub wiring: the clock and an awake clock (`pauserecord.MonotonicAwake` in production), a sleep that returns early on context cancel, the file watch (`internal/fswatch` over the prime's batten runs directory, each run's child status directory, the board dir's `board.json` and the hub's pause record directory), this machine's name and the owner read of the dobby's own beam (the prime owner for the prime), the binary check (`selfexec`'s `Check`), the probe interval (batten's `notice_probe_s`), the notice seam to the nearest live orch, and the seams the next cards add.
`Run` takes the lock at `geom.LockPath` without blocking and returns `ErrDobbyRunning` at once when another dobby holds it; it then reads its beam's owner and exits with a logged reason when another machine owns it, and reads it again on each probe tick.
On start it records its pid and start time in the state record; it keeps a sleep watch over the wall and awake readings of each tick (`pauserecord.SleepWatch`), records each sleep it sees in the state record, and the scheduling card owes every run a full pass on a sleep and on start.
It wakes on a watch event and on its timer, coalescing events to at most one tick a second; `nextWait` doubles the timer from the probe interval up to sixty seconds while no run is in flight and returns to the probe interval on any event or while a run is in flight; where the watch fails to open, as on WSL or through a junction, the timer alone drives it at the probe cadence (`PATTERN-daemon-wakeups`), and a periodic wake is never below one second.
Between ticks it calls the binary check, which re-execs onto a changed `lyx`; a re-exec is no start, so the state record keeps every owed full pass across it: `Run` reads a state record whose pid is its own as a re-exec, which owes no new full pass, and any other pid, or no record, as a start.
A tick's failure is logged once per episode and the loop goes on; a cancelled context, or SIGTERM through the verb's context, ends it after the tick in flight.
`cmd/lyx#unboundedWaitFiles` lists `internal/dobbyengine/loop.go` in place of `internal/battenshed/innerrun.go`, whose Run-Shed wait is gone, and `pattern/PATTERN-daemon-wakeups.md`'s "What counts" list names dobby's loop where it named batten's Run-Shed wait; the doc describes the loop, the lock, the owner exit and the wakeups.
Tests (tier0, fake clock, fake watch, temp state, parallel): `TestRun_LockOwnerAndWakeups` rows: a held lock returns `ErrDobbyRunning` and writes no state; an owner read naming another machine exits at start; an owner that moves to another machine on a probe tick exits then; a burst of five events in one second runs one tick; a sleep longer than a minute between ticks is recorded in the state; a watch that fails to open still ticks on the timer; a tick seam failing on three ticks warns once and logs its recovery once; a state record holding the dobby's own pid reads as a re-exec and one holding another pid as a start. `TestNextWait` rows: doubling to the sixty-second ceiling while idle, a reset on an event, the probe interval while in flight.

**ImpactSummary:** The engine's loop and its seams; nothing runs it until the beam verb wires it, and the wakeups guard lists its file.
