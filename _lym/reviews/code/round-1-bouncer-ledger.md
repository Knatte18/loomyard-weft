---
round: 1
ledger:
  - key: dirty-owned-file-reads-records-side
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-test-help-drops-plain-call-text
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: map-refusal-names-tags
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, all with local fixes the review text already specifies.
F1 is `dirty-owned-file-reads-records-side`, F2 is `gate-test-help-drops-plain-call-text`, and F3 is `map-refusal-names-tags`.
None gates, so the loop converges.
