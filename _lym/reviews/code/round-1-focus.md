---
round: 1
focus:
  - "internal/fabricengine/pushclassified.go: does the re-check loop stop at once, with no backoff sleep, on the code side when origin holds commits the local tip lacks, and does it push again only while origin's tip is contained in the local tip?"
  - "internal/fabricengine/pushclassified.go: does the loop stop on a landing, a deferral, a replay conflict, an anchor stop, an auth refusal and a transport failure, and rerun only after a bare rejection or an unclassified failure? Is it bounded by the length of `gitrepo.PushBackoffDelays()`, and does a persisting rejection still write exactly one `not_fast_forward` record and one mail?"
  - "internal/fabricengine/pushclassified.go and pushanchored.go: is `pushRebaseFreeRetryingOnce` fully removed with no caller or doc reference left, and does the records and bolt path reuse `retryWeftPush` and `retryBoltPush` rather than a new push path? Does any push force, rebase or merge?"
  - "internal/fabricengine/failedpush.go: does `recordsConflictWayForward` never say \"another machine\", and omit the `paths:` clause when no conflicting paths exist? Does the wording hold for both the records and the bolt side?"
  - "internal/fabricengine tests: does each acceptance criterion have a covering test, with the backoff sleep injected through `SyncOptions.BackoffSleep`, in the lowest fitting tier? Check the changed assertions in boltpullfirst_integration_test.go and pushanchored_integration_test.go for weakened coverage and for duplicates of existing push-class tests."
  - "internal/fabricengine/doc.go: does the push classification section state the new retry and wording, and match the code's behavior?"
---

Seed focus drawn from the decision record's scope, decisions and acceptance criteria.
