---
round: 2
focus:
  - "In internal/shedverbs/loopwait.go, deadLoopStop and loopExitStop build the cause and trace_file from the pid record's last child. Settle whether the child is named as in flight only when the status file still reads the record's producer and history length, and whether a moved status leaves the 'was in flight' clause out and keeps trace_file as the loop log path."
  - "In internal/shedverbs/doc.go, settle whether the wording about the in-flight child's producer matches the behavior of the stop after the change."
---

Focus targets the area changed to address the round 1 gating finding.
