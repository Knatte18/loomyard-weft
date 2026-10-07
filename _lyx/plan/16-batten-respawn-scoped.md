# Card 16 — batten re-spawns a running child whose spawn a dead batten confirmed

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/innerrun_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_test.go`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `internal/reedengine/directory.go`

**Intent:**
A restarted `lyx batten run` brings a running child back up as it was (D6, first bullet): a running child whose spawn was confirmed by an earlier, now-dead batten process is spawned again, and the child's bootstrap, idempotent against a live driver, takes the crash-resume path.

The spawn confirmation becomes scoped to the batten process that wrote it: `recordSpawnConfirmed` writes the writer's pid, and `spawnConfirmed` is true only when the marker names this process's own pid.
A marker from an earlier process, or one in the old `spawned` layout, reads as unconfirmed.
Batten's own run lock means no two batten processes watch one child at once, so a pid other than this process's is an earlier, dead one.

Two seams join `InnerRunDeps`, both resolved on Call like the others:

- `DriverStrand func(ctx context.Context) (ChildDriverStrand, error)` reports the child's driver strand in its reed state as none, live, retiring or dead, with `ChildDriverStrand` and its four values declared in `deps.go`;
- `ChildRunLockHeld func() (bool, error)` reports whether the child's run lock is held.

A nil seam resolves in `NewInnerRun`, the way a nil `Sleep` does: `DriverStrand` to none and `ChildRunLockHeld` to not held, so a caller building `InnerRunDeps` by hand (the batten lifecycle integration test) keeps today's behaviour.

In the running-and-unconfirmed arm, `Call` reads both before spawning: when the driver strand is live or the child's run lock is held, a driver is there, so it records the confirmation for this process and does not spawn, logging that it adopted the running child; otherwise it spawns as today.
So the re-spawn runs only for a child with no live driver strand and no held child run lock, and never stacks a second driver.
A read error from either seam is a returned hard error, like a status read error, and the next Call retries.
A halted or done child is still never re-spawned by this arm.

`battencli`'s wiring fills both: `DriverStrand` reads the task worktree's reed `Directory` (sessionless, card 4) through a small pure helper beside `driverAliveFrom`, matching rows with `loomengine.IsDriverStrand` (an absent task worktree, or no driver row, is none; a live row is retiring when its `Retiring` is set, live otherwise; a non-live row is dead); `ChildRunLockHeld` probes `shedrun.RunLock(taskLocation, shedrun.SelfRunID)` with a non-blocking try-acquire, releasing it at once when free.

`innerrun_test.go` pins: a confirmation written by this process skips the spawn; one from another pid, or in the old layout, with no live driver and a free lock spawns; with a live driver strand or a held lock it records the confirmation and does not spawn; a seam error is a hard error.
`wire_test.go` pins the helper's answers (absent pair, no driver row, live, retiring, dead), and `wire_seams_test.go`'s every-seam-filled list gains the two seams.

**ImpactSummary:** Batten's spawn marker changes content and meaning, and one arm of InnerRun gains two read seams; halted, awaiting and done arms are unchanged.
