---
round: 2
ledger:
  - key: tighter-read-rules-fixtures-untargeted
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: usage-entry-dropped-on-entry-time-attach
    rounds: [1]
    status: resolved
  - key: clean-converged-row-duplicates-verdicts-test
    rounds: [1]
    status: resolved
  - key: construction-refusal-outside-failure-table
    rounds: [1]
    status: resolved
---

Round 1 raised four keys.
Round 2 raised only F1, mapped to `tighter-read-rules-fixtures-untargeted`: round 1 missed fixtures for the review-side read and the usage record, round 2 found the ledger-side read (open BLOCKING under CONVERGED) breaks `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit`, which card 5 does not target.
Severity dropped from BLOCKING to MEDIUM as the missing set narrowed to one test.
The other three keys did not recur and are resolved.
