---
round: 2
ledger:
  - key: repeated-conflict-halt-no-way-forward
    rounds: [1]
    status: resolved
  - key: stale-conflict-record-routes-other-stuck
    rounds: [1]
    status: resolved
  - key: rerouted-round-inherits-hard-cap
    rounds: [1]
    status: resolved
  - key: repeat-check-comparison-unspecified
    rounds: [1]
    status: resolved
  - key: d1-rebegun-batch-wording
    rounds: [1]
    status: resolved
  - key: reroute-cap-halt-wrong-way-forward
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: last-routed-written-before-reroute-taken
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-record-existence-overclaim
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-bullet-list-swallows-decision
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's five findings no longer apply: round 2's focus verdicts find the repeated-conflict way forward, the hard-cap handling and the comparison key sound, and the record-lifecycle exposure is bounded by D3 and D4.
Round 2 raised four new findings.
F1 maps to `reroute-cap-halt-wrong-way-forward`: the Webster row now reaches shed's reroute-cap halt, which carries loom's shared parent-branch clause.
F2 maps to `last-routed-written-before-reroute-taken`: the adapter records a conflict as routed before shed decides to take the reroute.
F3 maps to `d2-record-existence-overclaim` and F4 to `d3-bullet-list-swallows-decision`, both local text corrections.
F1 and F2 change a decision in D3 and D5, so they need a fresh review.
