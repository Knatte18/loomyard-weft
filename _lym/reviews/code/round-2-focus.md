---
round: 2
focus:
  - "In internal/shuttleengine/run.go, DeliverPendingPrompt and the sendThroughPause, sendVerified and retryLanded path it calls: settle whether a pending-send record for the prompt text left by an earlier failed send, together with a later operator turn start past that record's offset, can make the send return nil without typing the prompt, and whether promptDelivered is then set only after the line was really typed."
  - "In internal/shuttleengine/hold.go and hold_test.go, attach_test.go: settle whether the covered guard, the turnStartRead guard and the send-failure fall-through of deliverPromptAtHold, and the send-failure branch and covering-pause handling of deliverPromptOnAttach, each have a test row that fails when the guard or branch is removed."
  - "In internal/shuttleengine/claudeengine/command.go writeOpenFiles and internal/shuttleengine/open.go Open: settle whether every error return after the session directory is created leaves no directory behind, and whether a test covers it."
  - "Settle whether the diff against the merge base touches any file outside the shuttle launch-delivery change, such as internal/fabricengine/recordswritelock_test.go, and whether the change description mentions it."
---

The focus entries follow the round 1 gating finding and its paired coverage gap, plus the open local corrections.
