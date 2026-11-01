# Dobby waits out a child's driver start before a dead-driver notice

Dobby sent `[dobby notice] …; driver strand is dead` for a go-driven child that was healthy.
`lym loom start` writes the child status as running at Preflight, and only then spawns the detached `lym loom drive`, which takes the driver lock as it starts.
A dobby tick that fell in that gap read the child as running with no strand and no lock, and `noticeCondition` reported a dead driver on the first such reading.

Two changes close the gap:

- `childDriverLockHeld` in `internal/battencli/dobbynotice.go` now counts the loom bootstrap lock (`loomengine.LoomBootstrapLock`) as alive, beside the driver lock and the run lock.
  The bootstrap holds it from before it spawns the driver until the driver holds its own lock.
- `noticeCondition` in `internal/dobbyengine/runnotice.go` reports a dead driver only after the child has been running for `driverStartGrace` (one minute) of awake time, counted from the `since` of its running episode.
  Machine sleeps and paused intervals do not count toward the grace, so neither can raise a notice earlier.

The `battenshed` and `dobbyengine` package docs describe both rules.

## Design decisions

- The lock check and the grace overlap by design: the lock covers the known gap exactly, and the grace covers a driver that is slow to start for another reason.
  A driver that never starts is still noticed, one minute late.
- `lym loom start` is unchanged.
  Taking the driver lock before writing the running status and handing it to the spawned driver would remove the gap at its source, but changes the spawn handoff; it is left out of this fix.
- After a machine takes a run over, the child's `since` dates from that machine's first sight, so the grace counts from there as well.
  A driver-dead notice the other machine already sent can therefore go once more, one grace later.

## Tests

- `TestChildDriverLockHeld` gains a held bootstrap lock case.
- `TestRunNotice_Conditions` gains a just-started child with no lock yet that sends nothing, plus a machine sleep and a paused interval that do not count toward the grace.
- The existing dead-driver tests advance the clock past the grace before they expect a notice.

Nothing needs a manual check.
