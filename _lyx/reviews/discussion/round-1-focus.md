---
round: 1
focus:
  - "D1 against Acceptance criteria: D1 prints `{\"issues\":[…]}` yet states an empty inbox as `[]`; settle whether the empty and non-empty shapes agree with each other and with today's output the orch and fork parse."
  - "D3: the fork-open rule says the orch starts no second fork when it knows one is open, while the same decision says an open fork is not named in the orch note; settle whether the orch has any way to know, or the clause is dead or contradictory."
  - "D4 attack surface: the fork gets `intake import`, close, fold and the label write of `upsert`; settle whether each write has a stated bound on what it can skip or let through, in particular a fold into an entry whose labels the fork rewrites and a close as noise that cannot be reopened."
  - "D2 and D4: the close-failure envelope drops `entry` and carries a `close` payload to re-run; settle whether D4's re-run rule and the Acceptance criteria describe the same envelope fields, and whether a re-run can loop or double-close."
  - "D5 and Constraints: settle whether the `ly:board` skill change and the stencil change leave no requirement stated only in `support-log.md`, such as the label-list-carrying `upsert` rule or the fold-label rule, that the plan writer needs."
  - "Scope against Decisions: settle whether every In item has a Decision or Acceptance criterion, and no Decision edits this task's own board entry."
---

Focus is derived from reading both artifacts against the rubric.
No entry targets a missing rejected alternative, a missing plan-writer notes section or call-site enumeration.
