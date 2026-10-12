---
round: 1
focus:
  - "D3 and D4, guard bound: does each Decision state what the guarded interrupt and the settled second Esc can still let through, and is that bound consistent with the orch watcher and pause engine paths?"
  - "D3, `ErrNoRunningTurn` as success: for callers that treat it as success (pause, orch watcher, planned `creel`), does any Decision let a session that was never interrupted be reported as stopped, and what bounds that?"
  - "D1 and D2 against D5, session classification: does every state/pane combination (running turn, `busy`/`background`, idle, failed capture) land in exactly one report list, and do the Acceptance criteria agree with D5's list definitions?"
  - "D4, retry loops: do the pause engine and the orch watcher both guarantee a maximum of two Escs per session and no Esc after a resume, and does each Decision say how that is bounded?"
  - "D6 and Constraints, resume: does the resume reason text in D6 hold for the case of a recorded session whose pane shows the picker, and does it match the `PATTERN-pane-text-injection` constraint?"
  - "Completeness before relocation: any proposal to move content out of `decision-record.md` must first check that the requirement or constraint it carries is kept in Decisions or Constraints."
---

Round 1 focus is set from a read of `decision-record.md` against the rubric.
The `Do not flag` items (missing plan-writer notes, rejected alternatives, call-site enumeration) and the stencil's ban on exact signatures and `file:line` citations are excluded from every entry.
