---
round: 1
focus:
  - "internal/fabricengine/pushanchored.go records-side retry loop: settle whether every exit path is bounded and returns the right error, including attempt == len(delays) and a nil or empty delays slice."
  - "recordsSideNotLanded in pushanchored.go: settle whether the order deferred-record check, then branchGoneOnOrigin, then ended message keeps a deferred push and a failed origin lookup from being misreported as ended or as retryable."
  - "branchGoneOnOrigin: read its definition and settle whether it separates a branch absent from origin from a transient lookup failure, and whether the lookup runs against the right remote and ref for the prime's records branch."
  - "Seen/Expected in the decision record: settle whether the change covers both causes the commit message names, a commit made during the push and a tracking ref left unmoved, and whether a transient classification can still fail the run at Seed-IDE."
  - "The retry loop's interaction with the weft-side absorbing push lock and the opts.BackoffSleep default: settle whether sleeping with the lock held, or reusing pushSide across attempts, breaks an invariant documented in doc.go."
  - "pushanchored_integration_test.go new subtest PrimeRecordsCommitDuringThePushIsPushedAgain: settle whether the post-receive hook reliably commits during the push, whether the test fails without the fix, and whether it duplicates coverage in the same test function; check the redundancy report entry for this function."
  - "doc.go and the pushanchored.go doc comments: settle whether each new claim matches the code and whether the touched comment edit in the test header replaced a duplicated line correctly."
---

Seed focus for round 1 of the review of the records-push change in `internal/fabricengine`.
The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
