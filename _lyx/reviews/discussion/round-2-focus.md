---
round: 2
focus:
  - "In decision-record.md D1, check what bounds who may issue a hub-wide `lyx pause`: the audience annotation in `internal/clihelp/annotations.go` only filters the command index, so settle whether the Decision states a real caller bound, its refusal with a way forward, and a matching Acceptance criterion."
  - "In D4 and D5, trace a covered run through the parent lock (held from Merge-In to Finalize per `internal/landingshed/doc.go`): settle whether the Decisions state what a covered waiter, a covered holder and the runs queued behind it can do during the pause, what the exception lets through, and how `lyx pause --status` shows it."
  - "In D7 and D3, check the orch interrupt against D3's grace and D6's rule that a line goes only into a session the entry owns: settle whether the watcher's timing, the report's orch case and the resume line's ownership condition are stated and covered by the Acceptance criteria."
  - "In D8, check how an operator moves a loom run that waits in the parent lock queue once the bare pause is gone: settle whether the named way forward (`lyx loom pause --after`, `lyx loom goto`) works for a row that cannot finish before it gets the lock."
  - "In D10, check what bounds the post-kill stop of shuttle runs: settle whether the Decision states how a run is tied to the killed step and what it must never stop, such as the loom driver's strand."
---

Entries name the questions to settle and carry no verdict.
