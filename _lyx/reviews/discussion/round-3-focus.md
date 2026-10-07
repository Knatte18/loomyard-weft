---
round: 3
focus:
  - "In decision-record.md, the Decision that bounds the status-side lock wait: does it state that the bound covers lock waiters only, what happens to a detached push child blocked behind a hung holder, and that the holder's git calls carry no timeout? Check that the Open risks entry and the support-log rejection rationale agree with what the bound actually buys."
  - "In decision-record.md, the Decisions on the new push entry's error return and on the `commit-records` envelope: when exactly one side fails, does the error name its side while `errors.Is(err, gitrepo.ErrPushRejected)` still holds, and does the envelope text stay true when only the code side failed? Check the acceptance criteria cover the single-side case."
  - "In decision-record.md, the Decision routing `lyx fabric push` normal mode through the new entry: after `Fabric.Commit` spawns a detached child, the in-process push contends with that child on the same lock. Is it decided whether the push waits bounded or without limit, and what an expiry returns? Check for a matching acceptance criterion."
  - "In decision-record.md, the Constraints entry on refusal messages and the Decision that rewrites the `commit-records` push-failure message: for a real divergence, is there a named way forward that actually succeeds, using no command `PATTERN-no-denied-recovery` forbids? Check that the Decision and the Constraint agree."
  - "For every new verb, flag, escape hatch or routing edge in decision-record.md, does the introducing Decision state what it can skip or let through and what bounds it (the attack-surface question)? Check any newly added text from this round's fixes the same way."
---

Entries target the open design and decision gaps and the consistency of the fixes against the rest of the record.
