---
round: 2
ledger:
  - key: skip-refusal-untested
    rounds: [1]
    status: resolved
  - key: nonfatal-failure-untested
    rounds: [1]
    status: resolved
  - key: build-comment-omits-once-only-rule
    rounds: [1]
    status: resolved
  - key: poisoned-message-names-only-copyhub
    rounds: [1]
    status: resolved
---

Round 1 raised four findings in `internal/hubforge`: two test gaps (skip refusal, non-fatal failure), a missing once-only-registration clause in the `Registration.Build` comment and doc, and a poisoned-template message naming only `CopyHub`.
Round 2's fresh review reports no findings, so all four are resolved.
