---
round: 3
ledger:
  - key: committed-tokencount-binary
    rounds: [1]
    status: resolved
  - key: reference-rule-test-missing-parallel
    rounds: [1]
    status: resolved
  - key: calibrate-peak-position-pricing-untested
    rounds: [1]
    status: resolved
  - key: batcher-cost-before-shadowed
    rounds: [1]
    status: resolved
  - key: reference-rule-test-redundant-row
    rounds: [1]
    status: resolved
  - key: readonly-command-quoted-name-blank
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
  - key: recover-batch-refusal-omits-readonly-route
    rounds: [2]
    status: resolved
  - key: accept-audit-usage-line-stale
    rounds: [2]
    status: resolved
  - key: tokencount-added-behaviors-untested
    rounds: [2]
    status: resolved
  - key: recover-batch-test-head-route-unpinned
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: accept-route-test-entries-hand-copied
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 1's five findings were not seen again and stay resolved.
Round 2's refusal message, usage line and tokencount findings are resolved: round 3 judged the message and usage line correct and found no tokencount gap.
The round 2 quoted-command-word finding narrowed to a first word that only contains a quote (`cat"x" y`), still open as LOW `design`.
Round 3 adds two LOW `scope` test-pin gaps: the HEAD-at-start phrase in the recover-batch refusal test, and hand-copied entry text in the accept-route tests.
