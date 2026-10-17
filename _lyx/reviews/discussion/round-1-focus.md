---
round: 1
focus:
  - "decision-record.md D4: does the premise hold that a `reminded` record carries its ids in `Unread` and no `ID`, and that matching `Unread` on every event kind cannot pull in an unrelated event? Check against `Store.Log` and `LogEntry` in `internal/creelengine`."
  - "decision-record.md D1 and D2: do the stderr line shapes (`creel: <done>, but its answer could not be written: ...` and `creel: <done>; <msg>`) agree with each other and with D3's prefix rule, and does every verb routed through `runVerb` have a defined line, including the ones that store nothing?"
  - "decision-record.md D2 and Scope: is the claim that a wait-timeout refusal holds no id, and that unconfirmed, expired and returned results do hold one, consistent with the acceptance criteria's exit codes 2, 3, 4 and 5?"
  - "decision-record.md acceptance criteria: does each Decision (D1 to D4) have at least one must-cover scenario, and does any criterion assert something no Decision establishes?"
  - "decision-record.md Open risks and Auto-mode assumptions: does any entry state a requirement the plan writer needs that lives only in support-log.md, per the completeness-before-leanness test?"
---

The decisions add no new verb, flag or escape hatch.
D1 carries an explicit bound for the newly checked write.
Rejected alternatives sit in support-log.md by design, so none of the focus entries asks for them in the record.
