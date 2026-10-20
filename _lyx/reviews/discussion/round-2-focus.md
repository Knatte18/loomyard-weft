---
round: 2
focus:
  - "Bolt replay in decision-record.md D5 and D3: check that the set-aside test and the replay match how Bolt stores entries (one shared board.json plus a README.md re-rendered with a minute stamp on every write), so two machines each holding a status commit of different entries replay without a stop. Settle whether any kept Bolt commit can be replayed without a README.md or board.json conflict."
  - "Definition of an ended entry in D3 and D6: check that an entry origin never held, for example one created while Bolt pushes failed, cannot be classed ended and torn down. Settle what separates removed from origin from never on origin."
  - "Wake in D4, D7 and D8: at wake every paused run's driver strand is still live and the hub pause is in force. Settle whether the record moves or pair-pauses such a pair before the hub pause lifts, and whether an ended entry with a busy driver is ever pointed at lyx resume."
  - "Reach of D5's ended set in the Fabric.Pull, push-retry and reconcile callers: settle where the set comes from per caller, and whether this machine's own landing records, from a run that entered Finalize, are ever set aside."
  - "D8 lyx cleanup: lyx cleanup without --apply is read-only. Settle what the reconcile does in that mode."
  - "Seams in D1, D4, D6 and D10: pairteardown, reedengine, creelcli and orchcli import fabricengine. Settle how the reconcile's teardown, liveness probe and push-failure notice are reached from fabricengine, and what each lock wait in D4 is bounded by given the parent lock can be held across a sleep."
  - "D10 push failure record: settle whether a failure episode that starts with no orch recorded is ever reported, what the deferral writes so the push holder neither loops nor goes silent, and where the gone class reads its state from."
---

Focus names where to look and which question to settle; it is no cap on what the reviewer may report.
