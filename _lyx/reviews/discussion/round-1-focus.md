---
round: 1
focus:
  - "D4 (Run-Shed waits without a time limit): the Bound names `lyx batten pause`, cancellation and the quiet notice. Settle whether each holds for every arm, including a wait whose child status file stays unreadable or unchanged while the driver strand is dead, and whether the driver-dead notice still fires from inside the wait."
  - "D5 stop-report lookup for a done child: 'the newest file in the run's drive-reports directory' can be an earlier stop's report. Settle whether the Decision bounds this so the notice never carries a stale report path."
  - "D6 removes the status file's modification time as the quiet signal and adds an api-error condition. Settle whether the Decision states what quiet can now miss (a live run with no readable transcript, a dead pid, several live runs) and what bounds that, beyond the no-live-run fallback."
  - "D7 removes the driver's 'run stopped' message for batten-watched runs. Settle whether a failure of the batten-watched decision (seed lacks the record, batten dies mid-run) can leave the orch with no notice at all, and whether the Decision names what bounds that."
  - "D3 removes a pending request as satisfied. Settle whether the `RequestedAt >= PhaseEnteredAt` test cannot drop a request the cycle did not cover (clock source, a request made just before the phase entry)."
  - "D8: settle whether the 'live process' check for an existing `batten:<slug>` window and the replace-if-exited rule can start a duplicate batten for one slug, and whether the Bound covers a window that outlives its batten."
  - "Consistency between D1 and D2 and the acceptance criteria: D1 says a found boundary qualifies whatever follows it, D2 keeps `ReadingTurnEnd` current only through the tick's event read. Settle whether the restart-in-`compacting` criterion and the 'no turn end after the boundary' criterion are both met by the stated mechanism."
  - "Consistency between D4 (every `Stuck` budget-exempt, 12-hour window and `max_bounces: 1440` rationale removed) and the Constraints that say shedengine's bounce and exempt-Stuck semantics are unchanged."
---

Seed focus for round 1.
Every entry is an attack-surface or internal-consistency question about a Decision, checked against the rubric's `Do not flag` list: none asks for rejected alternatives, a "Notes for the plan writer" subsection, or call-site enumeration.
The stencil tells the writer not to gather exact signatures or `file:line` facts, so no entry asks for them.
