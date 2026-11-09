---
round: 2
focus:
  - "internal/beamcli/dobbydeps.go startTestMap: when runproc registration of the spawned `lym gate testmap` child fails, does the code leave the child alone, as `PATTERN-child-processes` allows `runproc.EndProcess` only for orphans `lym tender reap` ends? Settle also whether the next tick can spawn a second unregistered child."
  - "internal/dobbyengine/loop.go tick: when the test-map step returns an error, are landBeam, announceLandings and the Deps.Tick hook still reached in the same tick, and is the error reported without being swallowed?"
  - "internal/orchcli/stop.go beamDown: on `lym beam stop` of the prime, is a running test-map refresh ended both when the dobby is live and when it is not, and is a failure of that end returned?"
  - "internal/beamcli startTestMap tests: do they pin the argv, the working directory, the `runproc.RoleTestMap` registration under `runproc.HubOwner` and the registration-failure behavior, at the lowest fitting tier without launching a test binary?"
  - "internal/dobbyengine/tidy.go countLandings and testMapStep: does the refresh count stay at 0 while `test_map_every` is 0, and does enabling the key later avoid an immediate refresh on a stale count?"
---

The focus covers the BLOCKING and gating design areas of round 1, plus the count behavior at `test_map_every` 0.
