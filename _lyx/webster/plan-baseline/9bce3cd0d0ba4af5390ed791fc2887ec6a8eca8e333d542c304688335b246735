# Card 8 — loom-start-replaces-retiring-driver

**Edit:**
- `internal/loomcli#driverStrandAction`
- `internal/loomcli#resolveDriverStrandAction`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli/bootstrap_test.go`
- `internal/loomcli/start_driver_test.go`

**Uses:**
- `internal/reedengine#StrandStatus`
- `internal/loomcli#reedDriverPaneProbe.RemoveDriverStrand`

**Intent:**
Implements loom's half of D7: `lyx loom start` replaces a live driver strand marked retiring instead of adopting it.

- `driverStrandAction` gains `driverStrandRetiring`.
  `resolveDriverStrandAction` returns it, with the strand's guid, for a found driver strand that is live and has `Retiring` set.
  A dead strand stays `driverStrandDead` whatever its mark, and a live unmarked strand stays `driverStrandLive`.
- `runDriverSpawnAndWait`, right after resolving the action, removes a retiring strand synchronously by guid through `driverPaneProbe.RemoveDriverStrand`.
  It then continues with the action set to `driverStrandNone`, so `mustSpawnDriver` and the stale-park-marker removal see no live driver and a fresh driver spawns.
  A failed removal releases the bootstrap lock and reports the error on the envelope like every other failure there.
  The fresh strand gets a new guid, so the old strand's detached remover is a no-op against it (card 7).
- The `start` help text names the retiring case.

Bound: start removes only a strand reed marked retiring, which some caller of `reed remove --detach` already asked to remove.
A live driver strand without the mark keeps today's handling in every case: working, parked (resume) and the not-parked refusal at a hand-back.

Tests:
- `bootstrap_test.go`: `TestResolveDriverStrandAction` gains rows for a live retiring strand (retiring action and guid), a dead retiring strand (dead action), and a live unmarked strand (live action, as today).
- `start_driver_test.go`: with a live retiring driver strand and a free run lock, `runDriverSpawnAndWait` removes that guid before starting a fresh driver through the starter seam.
  With a live unmarked strand it neither removes nor spawns.

**ImpactSummary:** One new driver-strand action handled at a single site in runDriverSpawnAndWait; the live, dead and absent arms are unchanged.

**Commit:** `8: loom start replaces a retiring driver strand`
