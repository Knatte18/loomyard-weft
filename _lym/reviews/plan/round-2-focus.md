---
round: 2
focus:
  - "Cards that change the drift judge's handling of an ended parent ask: check that both the diff-lines path and the diff-command path in internal/driftengine/verify.go carry the halt and the resume's re-ask before rulings are stored."
  - "Cards that add a new required seam or Paths field to a producer constructor: check that every test fixture and table row that builds that producer in its package is listed as an Edit target."
  - "Cards that add a wait inside a run's row: check the wait's wake interval and back-off against PATTERN-daemon-wakeups, and what a step-watchdog kill mid-wait leaves in the records."
  - "Cards that make an ask-again mail or stop a session under autonomy: check which strand the mail names and whether that strand is live when the mail is sent."
  - "Cards that clear a run record's session fields on goto: check that a live session is stopped first, and that the clear happens only after the move is written."
  - "Cards that force a setting under the run's autonomy: check that the Intent states what the forcing lets through and what bounds it."
  - "Cards that change behavior a file header or package doc.go describes: check that the file is an Edit target, not only a Uses entry."
---

Focus targets the shapes of defect found in round 1, stated without reference to its findings.
