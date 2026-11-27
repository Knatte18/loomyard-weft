---
round: 1
entries:
  - key: agent-carries-over-live-predecessor-instructions
    status: closed
    finding: F1
    reason: "a9d2832dc adds Spec.NoCarryOver, set by operatoragent.SessionSpec; roleKey returns the empty key for it, so newestPredecessor finds none (no carry-over block, record or notice) and the run record holds no role key; the StartSession row and TestSessionSpec assert it"
---
