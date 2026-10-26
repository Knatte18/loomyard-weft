---
round: 3
focus:
  - "Callers of `BeamLandRefusal` (`internal/battencli/beamland.go`): the pawl run's Finalize re-check, `lyx batten cancel <beam>` on a memberless beam, the dobby's can-land check and `lyx beam land`. The beam's orch runs from Beam-Up to Beam-Down. Settle whether the record states, once and for every caller, that the refusal judges members and member pairs and no longer blocks on a live orch or watcher, so none of these callers refuses by construction."
  - "Beam-Await accept in decision-record.md D9: the order of stopping the beam's dobby, writing the run status naming Parent-Align, and re-running the landing checks. Settle whether each of these holds: a retried or crash-re-entered accept leaves the beam's dobby live when its re-check fails; no window exists in which `lyx beam start` or a member's hand `lyx batten run` restarts the dobby with no hold standing; the re-check does not include a check the accept's own status write makes false."
  - "Guards the beam entry loses or weakens at Beam-Await: the board run lock on the beam project entry, the Beam-Await driver's respawn behavior in `internal/battenshed/innerrun.go` and its driver-dead notice, and the cancel refusal's read of local Bolt. For each, settle whether the Decision that lowers or bypasses it states what it now lets through and what bounds it."
  - "D5's mixed-version bound against `normalizeTask` and `checkProjectFields` in `internal/boardengine` (task.go, replay.go): settle whether the stated behavior of an older build matches what those functions do with a project entry carrying `recipe: pawl`."
---

Focus for round 3 targets the new text written to close round 2's gating findings.
Every entry is a question to settle, and none caps severity.
