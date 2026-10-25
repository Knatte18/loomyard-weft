---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md`, D6 and D7 (the Drift-Verify row and its scope-full bound): D2 promises that under scope `full` no drift is accepted before the parent rules. Settle whether that promise holds at every entry of Drift-Verify, not only at the end of an attempt: a session that accepts drift and then ends in an error, transient or interrupted stop leaves its acceptance in the scope record's accepted list, and the resumed row compares first with the accepted tests skipped. Check that the record states what the resumed row does with an acceptance the parent has not ruled on, and what bounds that."
  - "In `_lyx/discussion/decision-record.md`, D2 (gate paragraph), D5 (flag-off paragraph) and the acceptance criteria: `Drift-Await` carries `entry_gate: drift.enabled`, and `shedengine.Shed.enter` evaluates a gate only when `EnteredRow` does not already name the row, while `lyx loom goto` sets `EnteredRow` to its target. Check that every claim about turning the flag off mid-run is true for a row not yet entered, an entered and waiting row, and a halted row, and that an operator who gives up regression at an entered row has a named way forward."
  - "In `_lyx/discussion/decision-record.md`, D5 and D6: check that one predicate defines the capture `Drift-Verify` counts as passed, and that the halt on no passed capture, the precondition of `lyx drift waive` and the acceptance criteria all use it, so the halt that names waive is a halt waive admits."
  - "In `_lyx/discussion/decision-record.md`, D4: check that the record settles, rather than leaves to the plan, whether a step child can reach `spawnLoop` in `internal/shedverbs/loopwait.go`, the one other non-test `proc.DetachBreakaway` caller, and that the compare marker's liveness test and write point relative to the gate slot wait are stated."
---

The focus names where the round 3 findings were fixed, so a reviewer checks the new text against the code it cites.
