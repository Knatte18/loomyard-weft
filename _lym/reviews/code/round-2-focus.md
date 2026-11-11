---
round: 2
focus:
  - "internal/webstercli/ownership.go, orphanedMasterRefusal and refuseOrphanedMaster: does a lock-probe error (RunActive failing) still produce a refusal carrying run_ended true, and does the refusal's way forward (re-run the verb) agree with what the Master stencil tells the Master to do on that key?"
  - "internal/webstercli/cli.go addVerbs and ownership_test.go: does a test run each of the six bracket verbs (begin-batch, await-batch, record-batch, recover-batch, fix-verify, fix-plan) through the real verb tree with a free run.lock and a Master strand name, and show a non-bracket verb is not refused for that reason?"
  - "internal/websterengine/verifygatereport.go renderVerifyGateFindings: for a failed verify with no named failing test (Failures empty, LogTail set), does the rendered text avoid an empty Failing identities list and a card hint that does not apply?"
  - "internal/websterengine/verifygate.go stuck-reason code (failedEntryReprompts, cutHead, cutTail, the per-arrival command reset, verifyGateStuckFailure): do the tests in verifygate_test.go have a row that fails if each branch breaks, and does an empty output render without a dangling output tail clause?"
  - "docs/overview.md decision-add description and the Long help of the decision verb in internal/loomcli/decision.go: do both list notice_held_back alongside notified and notice_errors?"
---
