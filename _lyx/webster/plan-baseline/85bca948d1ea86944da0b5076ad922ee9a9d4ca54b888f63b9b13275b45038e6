# Card 18 — loom-idle-release-pause

**Edit:**
- `internal/loomcli#idleReleaseOn` -> `func idleReleaseOn(locks *parentlock.Store, self func() parentlock.Owner, agents func() ([]parentlock.AgentState, error), bound time.Duration, awake func() time.Duration, covered func() bool) func()`
- `internal/loomcli#loopIdleRelease` -> `func loopIdleRelease(locks *parentlock.Store, self func() parentlock.Owner, agents func() ([]parentlock.AgentState, error), bound time.Duration, covered func() bool) func()`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#TestLoopIdleRelease`

**Intent:**
Keep a covered run's parent lock through the pause in loom's own loop (D5): the loop's idle release, which `OnWake` runs on every watch wake, must not count paused time as idle time.
`idleReleaseOn` takes `covered`; on a wake where it reports true it restarts the idle watch and returns without releasing, so the lock stays held until the resume and the bound is measured afresh after it.
Other runs' lock-taking rows keep queueing behind the holder as for any live holder.
`loopIdleRelease` passes `covered` through and swaps its `time.Since` awake reading for `pauserecord.MonotonicAwake()`.
`specFor` builds `covered` as a read of `pauserecord.ReadCoverage` at `hubgeom.PauseGeometry(c.location)` for the run's slug; a read error reads uncovered with one Warn.

Test: `TestLoopIdleRelease` gains a `covered` field over its wakes and two rows: a holder whose agents wait past the bound while every wake is covered keeps its record; one covered for the first wakes and uncovered after keeps its record until the bound passes again from the first uncovered wake, then releases. The existing rows pass a `covered` that reports false.

**ImpactSummary:** Two loomcli-internal functions re-signed with their one production caller, specFor, and their one test on this card.
