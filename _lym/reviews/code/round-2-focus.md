---
round: 2
focus:
  - "internal/dobbyengine/land.go `landBeam`: when `deps.Land` returns a refusal that only waits on run state, such as the pawl run not yet at Beam-Await, does the next tick call `Land` again, and is a refusal notice posted at most once per episode? Check `recordLand`, `LandState` and the `TestLandBeam` rows."
  - "internal/beamcli/dobbydeps.go `beamLandReading` and `deps.Land`: is an unflagged beam read as not passing before any other read, and does `deps.Land` write the same land request as `lym beam land` and return its refusals? Check that a test in internal/beamcli reaches each."
  - "internal/beamcli/deps.go `MarkReady`: after a failed commit following the record write, does rerunning the verb commit the already written record, or does it report `already` with the flag untracked? Check the matching `TestMarkReady` rows."
  - "internal/beamcli/land_test.go and internal/loomcli/pawlwiring.go `pawlRecheck`: do tests pin the ready refusal's position after the owner checks and before the work-remaining refusal, the `Ready` and `MarkReady` error ways forward, and the unflagged line's position in the re-check lines?"
  - "internal/dobbyengine/loop.go `tick` doc comment and internal/loomcli/doc.go Beam-Await re-check paragraph: do they match the landing step and the ready record the re-check now reads?"
---

Each entry names a site to settle; none presumes a verdict.
