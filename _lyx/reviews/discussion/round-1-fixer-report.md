# Fixer report — discussion, round 1

All five findings are fixed as proposed, in `_lyx/discussion/decision-record.md`.

## Changes

- F1 (MEDIUM): D4's guid bullet now covers every method that consults an engine with neither a guid nor a run, `InterruptSettle` included, resolving the engine from that run's record.
  It names `tenderengine`'s settle as passing each paused session's guid and `orchcli`'s runner session as passing the orch run's guid.
  Added the acceptance scenario that `InterruptSettle` on a run of the non-default fake answers that fake's settle.
  R1's "event-only methods" now reads "guid-less engine-consulting methods" to match.
- F2 (MEDIUM): D7 gains a Bound: `websterengine` audits and parses (`TurnEndedAfter`) through the default engine, which is the engine that ran its sessions only because no Webster spawn spec names an engine in this task.
  New R4 tells `engine-model-choice` to keep Merriam and batch specs on the default engine or resolve Webster's audit and parse engine from the batch run's record.
- F3 (MEDIUM): D4's Attach bullet now has an attach check the spec's `Requires`, including `ForkSubagents`' implied `forks`, against the recorded engine and refuse with D8's error, leaving respawn as the way forward.
  Added the matching acceptance scenario.
  D8 is aligned with it: its heading, a line pointing to D4's attach check, and its Rationale.
  The Scope's `Spec.Requires` bullet also says the list is checked at `Start` and attach.
- F4 (LOW): R2's first clause now covers only runs started before this task lands, whose records have no `engine` key; the clause on long-lived set holders is unchanged.
- F5 (LOW): Added the acceptance bullet that the Claude engine's `InstructionFiles` answers `CLAUDE.md` and `CLAUDE.local.md`.

## Disputed

None.

## Deferred

None.
