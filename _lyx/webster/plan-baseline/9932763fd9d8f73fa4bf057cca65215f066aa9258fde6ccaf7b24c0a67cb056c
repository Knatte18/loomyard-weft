# Card 19 — watchdog-awake

**Create:**
- `plan:internal/shedverbs#fakeClock.Awake` -> `func (c *fakeClock) Awake() time.Duration`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#idleWatch`
- `internal/shedverbs#watchIdle`
- `internal/shedverbs#waitForChild`
- `internal/shedverbs#fakeClock`
- `internal/shedverbs#TestLoop_WatchdogKillsAnIdleChild`
- `internal/loomcli#loopSpecFor`
- `internal/loomcli#TestSpecFor_FillsRunIdentityForSelf`
- `internal/shedverbs/doc.go`
- `internal/shedverbs#shedverbsAllowedImports`

**Intent:**
Make the shed loop's step watchdog measure idle in awake, unpaused time and never kill while a pause covers the run (D5, D9).
Its reading today is wall time since the newest trace-file mtime or agent activity, so a wake after a sleep longer than its window kills at once, and a pause longer than the window would kill too.

- `LoopSpec` gains `Awake func() time.Duration` (nil selects `pauserecord.MonotonicAwake()`) and `Pause func(now time.Time) (pauserecord.Coverage, error)` (nil never covers); `idleWatch` gains both.
- `watchIdle`, on every wake, feeds a `pauserecord.SleepWatch` the clock's wall and awake readings and reads the coverage; an error reads uncovered with one Warn.
  While covered it judges nothing and keeps waking on its backoff.
  Otherwise the idle time is `pauserecord.AwakeSince(newest, now, sleeps, coverage.Intervals)`, and the child is killed only when that reaches the window; the wake schedule keeps the one-second floor and one-minute ceiling.
- `waitForChild` hands both seams to the watch.
- `shedverbsAllowedImports` gains `github.com/Knatte18/loomyard/internal/pauserecord`.
- loom's `loopSpecFor` sets `Pause` to a read of `pauserecord.ReadCoverage` at `hubgeom.PauseGeometry(loc)`'s record path for the run's slug, and leaves `Awake` nil.

Bound: the watchdog still kills a child idle for its window of awake, unpaused time; a pause or a sleep only delays the kill.

Tests: `fakeClock` gains an `awake` reading that `Sleep` advances with wall time and an `asleep(d)` step that moves wall time alone; `TestLoop_WatchdogKillsAnIdleChild` passes its `Awake` and gains rows: a three-hour sleep inside a ten-minute window leaves the child alive until ten awake minutes pass, then kills it reporting 10m0s; a child covered by a live entry for two hours past its window is never quit or killed, and halts on its own once the script releases it; an entry covering an hour then resumed kills the child only ten unpaused minutes after the resume.
`TestSpecFor_FillsRunIdentityForSelf` writes a live entry for the run's slug to the location's record and asserts `spec.Loop.Pause` reports it covered.
`doc.go` describes the awake, unpaused idle reading.

**ImpactSummary:** Two optional LoopSpec seams read only by the step watchdog; every arming's watch becomes sleep-aware through the nil Awake default, and only loom sets the pause seam.
