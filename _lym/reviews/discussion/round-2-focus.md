---
round: 2
exclude_lenses: []
focus:
  - "Crash recovery in decision-record.md D12: settle whether the strand identity of an attempt is fixed and recorded in state before the spawn, so that re-entry looks the run up by that recorded identity and never spawns a second agent on the tree; shuttleengine.FindRun resolves only a guid the caller already holds."
  - "Batch list handed to the shared post-batch checks in D3 and D11: settle whether the checks (unbegunCards, laterDeleteReferenceReasons, isLastBatch in internal/websterengine) see the pending cards and a correct last-batch flag when Weaver records only one batch per partition."
  - "Active-implementer seam in D1 and D13: settle what decides which implementer is active for the plan gate and the D1 guard, and what a missing or corrupt state file of either implementer lets through."
  - "State shape in D11 against D3, D9 and D12: settle whether the shared checks take Webster's State, BatchState and Geometry or a narrow input, and whether the record leaves this to Plan."
  - "Strand lifecycle in D6, D7 and D12: settle whether a failed or timed-out attempt's strand is stopped before the next attempt or the Stuck return, and whether re-entry can attach to a live stale strand."
---

Every lens ran in round 1 and the BLOCKING and MEDIUM design findings span all four, so none is excluded.
