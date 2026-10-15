---
round: 2
ledger:
  - key: d5-held-span-kind-double-count
    rounds: [1]
    status: resolved
  - key: d5-carrier-set-misses-burler-review-gate
    rounds: [1]
    status: resolved
  - key: d7-open-rework-visit-attribution
    rounds: [1]
    status: resolved
  - key: d4-child-number-undefined-for-review-segment-rows
    rounds: [1]
    status: resolved
  - key: d4-examples-contradict-d3-rule
    rounds: [1]
    status: resolved
  - key: auto-mode-assumption-restates-d5
    rounds: [1]
    status: resolved
  - key: earlier-entries-regate-spans-during-hold
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: support-log-keeps-retired-4-7-example
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's six findings are resolved: round 2's review re-checked D3 through D7 and raised none of them again, and its focus verdicts confirm the carrier set, the open visit and the numbering examples.
Round 2 raised two new keys.
The gating one is a MEDIUM design finding: entries listed before a held entry gain a gate span per recheck during the hold, which D5 and D6 leave unbounded.
The other is a NIT: the support log still cites the retired `4.7` example that D4 replaced with `6.5`.
