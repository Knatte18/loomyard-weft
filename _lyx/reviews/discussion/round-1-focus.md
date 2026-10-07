---
round: 1
focus:
  - "D3 and Constraints (Windows) against the acceptance criterion 'no process of its group survives': does a Windows new process group actually give a whole-tree kill, or does the criterion and Decision promise more than the Windows fallback delivers?"
  - "D5 bound: a non-live strand with all outputs present but its gate unpassed is dropped (Open risks, last item). Does D5's stated bound cover what this drop lets through, or is a guard on 'finished' weakened without a bound?"
  - "D6 and D7 consistency: the states each handles (running, halted, awaiting, parked, not yet parked, no live driver) and the way-forward commands each names. Do they agree, with no state left unhandled or contradictorily routed?"
  - "D2 callback and D1 hook order: do the Decisions state what a stale or wrongly targeted wait mark (driver strand versus agent strand, Go-driven run) can show or hide, and what bounds it?"
  - "Goal versus Scope and Decisions: every clause of the Goal has a Decision and an acceptance criterion, in particular 'revives only strands with work left' for strands that carry no done-when list."
  - "D8: does any Decision introduce a cross-cutting rule (done-when list on strands, pane-option wait mark) that CLAUDE.md says belongs in PATTERN in the same commit, contradicting 'no PATTERN entry is added'?"
  - "Relocation pass over decision-record.md: deliberation or history that carries no requirement for the plan writer and belongs in support-log.md, after the completeness-before-leanness check."
---

Seed focus only; the reviewer still reads both files in full.
Excluded per the rubric: a missing optional plan-writer section, missing rejected alternatives in `decision-record.md`, and call-site enumeration.
The `Notes for the plan writer` paths are optional head-start material and are not a focus item.
