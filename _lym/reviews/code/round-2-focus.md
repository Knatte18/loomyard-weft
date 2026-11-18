---
round: 2
focus:
  - "Check in internal/fabricengine/bolt.go (Bolt.Pull, Bolt.PullThenCommitWritten, pullFirst) and internal/fabricengine/pushanchored.go (PushPairAnchored) whether pauserecord.Admit with the push-lock operation is passed before the board write lock and before the pair's teardown lock are acquired; settle whether any path waits at the gate while holding a shared lock."
  - "Check that each gate site in lockWeftForReplay (internal/fabricengine/pull.go), the Bolt pull path (internal/fabricengine/bolt.go), retryBoltPush (internal/fabricengine/pushclassified.go) and takeoverBranch.land (internal/fabricengine/takeoverwrite.go) has a test that fails when its Admit is removed or moved after the lock."
  - "Check the claim in _lym/landing/summary.md that dobby needs no change for a paused pair against internal/tenderengine/pause.go, where Pause writes its live entry before it lists sessions, and against dobby's PauseCovers, planPass and readyToStart."
  - "Check that the commit range carries a Loosened-Check trailer for rows removed from TestPushDeferral_PutsBackTheUnpushedFlags in internal/fabricengine/coalesce_test.go, and that the pushDeferral doc in internal/fabricengine/pendingflags.go and doc.go state when the deferral fires, given that coalesceFlagged admits before every acquire."
---

Prose and test-coverage findings from round 1 outside these questions are left to the ordinary review.
