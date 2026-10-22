---
round: 1
focus:
  - "D3 and D4 ordering: D4 step 2 takes batten's run lock, yet step 5 waits for the run lock after the end and the survivor rule names a run lock still held. Settle whether the lock sequence is coherent and whether D3 step 4's 'leaves locks held' matches it."
  - "D2 unreachable-origin path: the verb proceeds with a Warn and reads ownership from the local board. Settle whether the Decision states what this can skip (a run owned by another machine, a stopped branch) and what bounds it."
  - "D1, D2 and D4 attack surface: the verb ends a live run mid-turn from the orch, and `--remove` force-removes with a snapshot. Settle whether every Decision that introduces a skip or a removed guard states its bound (D11's hand-back row, D4's skipped quiet wait and refusal probe, D5's forced removal)."
  - "Deferred facts: check D4, D8, D10 and the Open risks for a fact the design rests on that Plan or an implementer is left to find (the index.lock risk, older-binary processes, that `Lift` sends no resume line with no sessions, that the removal mode leaves the claim to the caller). Settle whether each is researched with its evidence recorded."
  - "Relocation and leanness: check Scope, Constraints, Auto-mode assumptions and Open risks for deliberation narrative that belongs in support-log.md. Before any relocation finding, confirm no requirement the plan writer needs would be lost, and extract it into Decisions or Constraints first."
  - "Support-log consistency: Interview Turn 3 still records picks (refuse on an unreachable origin) that D2 and the ledger's Q10 revise. Settle whether the log misleads a reader about the final design."
---

Seed focus from one read of both files and the discussion spec.
Not promoted, per the rubric's `Do not flag` list: missing rejected alternatives in the record, an absent plan-writer note section, and call-site enumeration.
The board-entry-edit bullet does not apply: the Decisions clear a run's claim status, which the rubric leaves unflagged, and the Scope's Out list excludes the entry's body and scope.
