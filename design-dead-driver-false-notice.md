# Dobby sends a dead-driver notice for a run that has just started

`dead-driver-false-notice` · Task · bug, dobby, loom · running · Code-Burler-Fix

## Seen (hub orch, 2026-11-01)

Dobby sent `[dobby notice] pr-continue-wakes-row: running at Preflight; driver strand is dead; since 2026-11-01T06:18:50Z; history 0` (creel #183) a second after the child run started; the go-driven darn run was healthy and reached Darn-Write moments later.

- `noticeCondition` in `internal/dobbyengine/runnotice.go` reports a dead driver the first time `DriverAlive` returns false, with no grace and no start-up window; `memory.Since` is only the episode stamp.
- `noticeDriverAlive` and `childDriverLockHeld` in `internal/battencli/dobbynotice.go` count a child alive only through a live driver strand, a held `DriverLock` or a held `RunLock`, never the bootstrap lock `lym loom start` holds.
- `lym loom start` (`runDriverSpawnAndWait` in `internal/loomcli/start.go`, with `internal/loomcli/drivewiring.go`) writes the child status as running at Preflight with history 0 first, and only then spawns a detached `lym loom drive`, which takes `DriverLock` as it starts.
  In between, the child reads as running with no strand and no lock.
- Times: child status episode 06:18:50, creel #183 at 06:18:50.99, Preflight done 06:18:51.
  A dobby tick fell inside that sub-second window; `batching-effect-audit` passed the same window with no tick in it and got no notice.

## Idea

Dobby calls a driver dead only once the child has had the chance to start one.
The smallest fixes: count the bootstrap lock `lym loom start` holds as alive beside the driver and run locks, or require a dead reading to hold across two ticks or a short configured grace from `memory.Since`.
A cleaner one: `lym loom start` takes the driver lock before it writes the running status and hands it to the spawned driver.
A test covers a child running with history 0 and no lock held yet, and asserts no notice.
