---
round: 1
focus:
  - "internal/dobbyengine/land.go `landBeam`: settle whether the Land seam's refusal and the posted notice match the parent's Option B entry, which says dobby lands a flagged beam itself and posts only when it cannot; check the episode state is recorded once, resets when the reading stops passing, and survives a dobby restart."
  - "internal/beamcli/dobbydeps.go `deps.Land` and `beamLandReading`: settle whether dobby's call of `land` writes the same land request `lym beam land` writes, whether a refusal inside it (owner, run, flag, work remaining) is reported rather than swallowed, and whether an unflagged beam reads as not passing before any other read."
  - "internal/beamcli/land.go and cli.go: settle that the unflagged refusal sits after the owner checks and before the work-remaining refusal, names `lym beam ready <slug>` then `lym beam land <slug>`, and that `lym beam ready` refuses a non-landable entry, commits through fabricengine, and reports an already-flagged beam without writing."
  - "internal/pawlengine/ready.go and internal/loomengine/config.go: settle that the record is read and written through internal/state, that a record naming another slug or an absent record reads as not flagged, and that the record and lock paths mirror `_lym/landing` and `.lym/landing` through lymdirs."
  - "internal/loomcli/pawlwiring.go `pawlRecheck`: settle that the Beam-Await re-check adds the unflagged line using the beam's own slug and yields no lines for a flagged beam with nothing else wrong."
  - "Test coverage: the rename of landnotice_test.go to land_test.go dropped and rewrote tests; settle that every behavior of the old notice episode logic the diff still carries, and each new behavior (Land success, Land refusal notice, parent-owner notice, unflagged no-op), has a test, and that none duplicates another."
  - "Docs: settle that the package docs of beamcli, dobbyengine, loomcli and pawlengine and docs/overview.md describe the flag, the verb, the dobby landing step and the refusals as the diff now behaves, with no leftover wording about a can-land notice as the dobby's only action."
---

Seeded from the board entry, the decision record with its Option B amendment, and a read of the diff's dobby, beamcli, pawlengine and loomcli changes.
