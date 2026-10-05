---
round: 2
ledger:
  - key: prefix-head-lifecycle-unstated
    rounds: [1, 2]
    status: open
  - key: delete-contract-files-no-way-to-finish-run
    rounds: [1, 2]
    status: open
  - key: transcript-evidence-counts-recorded-not-successful-writes
    rounds: [1, 2]
    status: open
  - key: d3-fallback-unbounded-choice-left-open
    rounds: [1]
    status: resolved
  - key: d7-enumeration-belongs-in-notes
    rounds: [1]
    status: resolved
  - key: d8-sequence-rule-deferred-to-plan
    rounds: [1]
    status: resolved
  - key: d10-scan-prohibition-vs-recommendation
    rounds: [1]
    status: resolved
  - key: d3-rationale-narrative-sentence
    rounds: [1]
    status: resolved
  - key: d3-prefix-vs-permission-rule-matching
    rounds: [2]
    status: open
  - key: r4-attributes-acceptance-to-absent-operator
    rounds: [2]
    status: open
---

Round 1 raised eight findings; round 2 raised five.
Five round-1 keys were not raised again by the fresh review and are marked resolved.
Three round-1 keys reappear in narrower form: the pre-fix head lifecycle (R2 F1, reset and `--fresh`), the contract-file way forward (R2 F4, re-entry condition for `lyx webster run`) and failed-tool-result evidence (R2 F5, acceptance criteria).
Two keys are new: D3's effect on allow and deny matching (R2 F2, the only gating finding, `design` MEDIUM) and R4's attribution of acceptance to an operator who did not answer (R2 F3).
