# Card 14 — orch-correction-guidance

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:** Have the orch send a scope correction through `lyx loom decision add` whenever the run has a record, darn included, and expect darn's PR-Gate as an alignment escalation (D11).

In `orch-template-role.md`:
"The run loop" item 1's darn sentence keeps that the Darn-Review perch reviews the writer's diff before Publish unless `review_max_bounces` is 0 and that "a reject re-spawns the darn writer with the findings", and replaces "PR-Gate is the operator's review point" with PR-Gate reaching you as an alignment escalation, as in loom.
"Fixing a run" says a scope correction on a run whose decision record exists goes through `lyx loom decision add <slug> --by parent` even while an agent of the run is alive, and also in a message to that agent; the line that keeps `decision add` for when no agent of the run is alive keeps that rule for `lyx loom goto` alone.
"The board"'s refusal bullet drops "or any darn run" from the runs the verb refuses on for want of a record, keeping the board-note fallback for a run with no record yet; the next bullet adds that a darn run's verb refuses until Darn-Write has committed its record, and then the decision goes by message to the Darn-Write session.

The `orch-template-role.md` row of `wordingClaims` gains a must claim on the alignment-escalation wording in "The run loop", a mustNot claim on `PR-Gate is the operator's review point`, a must claim on `even while an agent of the run is alive` in "Fixing a run", a mustNot claim on `or any darn run` in "The board" and a must claim on the Darn-Write message fallback there.

**ImpactSummary:** orch guidance and its wording claims only; no Go changes.

**Commit:** `14: the orch records a correction through decision add whenever a record exists`
