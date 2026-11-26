---
round: 1
entries:
  - key: resumed-recovery-loses-question-and-answer
    status: closed
    finding: F1
    reason: "6f179d83f and b2429637d add --answer, refuse a blank one before any spawn, persist need and answer in ParentAnswers, render them in the recovery prompt, update Master's waiting and re-entry rungs, and TestRecoverBatch_WaitingResume pins both"
---
