---
round: 1
focus:
  - "`RecoveryRetry` in `internal/websterengine/recoverbatch.go`: does the new zero-count early return run after the kind, terminal, dead and cap checks, and does it leave the one-counted-recovery commit rule and the empty-start false unchanged?"
  - "Callers of `RecoveryRetry` (`status`, `recover-batch` terminal envelope, `begin-batch` remedy): can any dead recovery record at zero counted recoveries that is not a reset batch, such as one whose spawn counted nothing, now loop or retry where the old rule refused?"
  - "Existing tests whose dead-recovery fixtures had `Recoveries` 0 and expected false (`recoverbatch_test.go`, `beginbatch_test.go`, `internal/webstercli/cli_test.go`): were they updated to the new rule only where the decision record licenses it, and is the 'record with no start' row still meaningful?"
  - "Scope against the decision record: do the `existingReportRemedy` wording in `beginbatch.go`, the Master stencil, `doc.go`, the `recover-batch` long help and the `RecoveryStartSHA` comment in `state.go` all state the new rule consistently, and does the diff leave the reset verb's state writes and the recovery cap untouched?"
  - "Stencil `webster-template-master.md`: does the changed wording state that `recovery_retry: true` covers a batch whose count a reset cleared, without dropping or weakening any other dead-rung rule?"
  - "Test economy: do the three new test rows (engine, begin-batch, status) each cover a distinct reader of the verdict, or does one duplicate another's coverage?"
---

Seed focus from the board entry and decision record; the diff is the 9-file change over `ed98ccf71`.
