---
round: 1
ledger:
  - key: card3-no-tail-rows-unreachable-conditions
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-gate-dismissal-gap-duplicate-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card1-lone-eol-mark-first-send-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card4-stale-testtiming-keep-reason
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised four findings.
F1 (design, MEDIUM) is the only gating one: card 3's named no-tail rows run on `skillReed`, which serves an empty startup capture, so they pass through the empty-capture exclusion and never test the `Started` or cause checks.
F2 (LOW) is a test-economy overlap between card 4's new gate-dismissal test and an existing gate-never-clears test.
F3 (LOW) is card 1's fake reed serving the lone-`%` box only to the first send.
F4 (NIT) is a stale `testtiming:keep` reason on card 4's rewritten test.
