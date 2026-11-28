---
round: 2
focus:
  - "internal/landingshed/finalize_test.go against internal/landingshed/finalize.go (landUnderLock, pushParent): with Deps.LandLocally wired, does a test fail when only the post-merge head read errors and the error check is removed? The step-1b4b read before the merge carries the same message, so settle which read the injected error reaches, and whether LandLocally is shown not to be called."
  - "internal/loomcli/landingdeps.go (recordHeldLanding, writeLedgerLanding): can any failing step between the status checkpoint commit and the ledger landing record leave that commit free for a pair push? Settle where the parent pair's branch is read relative to HoldPairPushLock and commitRunStatus, and whether writeLedgerLanding's signature, its doc comment, its callers and the signature and design bullet in _lym/landing/summary.md agree with each other and with the amended board entry."
  - "internal/landingshed/deps.go (LandLocally contract) and internal/landingshed/finalize.go (the comment on the post-merge head read, the recordLanding and pushParent doc comments, the Finalize.Call doc): does each describe the tip correctly on a re-run over a parent that already holds the run's work, possibly with another run's commit on top?"
  - "internal/loomcli/landlocally_integration_test.go, TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed: can each remaining assertion fail for a writer that records the parent's head instead of the tip it is given, and do the test's name and doc comment match what it checks?"
---
