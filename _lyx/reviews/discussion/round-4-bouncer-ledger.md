---
round: 4
ledger:
  - key: decision-add-unbounded
    rounds: [1]
    status: resolved
  - key: accept-on-budget-unbounded
    rounds: [1]
    status: resolved
  - key: circling-at-spent-budget-cause-unstated
    rounds: [1]
    status: resolved
  - key: circling-guard-weaker-than-definition
    rounds: [1]
    status: resolved
  - key: per-key-rounds-source-unstated
    rounds: [1]
    status: resolved
  - key: d1-d4-budget-limit-disagree
    rounds: [1]
    status: resolved
  - key: parent-notice-content-unbounded
    rounds: [1]
    status: resolved
  - key: d8-bound-reader-undecided
    rounds: [2, 3, 4]
    status: open
  - key: d4-budget-test-off-by-one
    rounds: [2, 3, 4]
    status: open
  - key: circling-continue-hits-shed-block
    rounds: [2, 3, 4]
    status: open
  - key: repeated-continue-no-ceiling
    rounds: [2]
    status: resolved
  - key: d7-accept-bound-loose
    rounds: [2]
    status: resolved
  - key: pinned-failing-smoke-tally
    rounds: [2]
    status: resolved
---

Round 4 raised three findings.
F1 (design, BLOCKING) maps to `d4-budget-test-off-by-one`: D4 names several counts and budgets for one test, and the `Bounces` seam's budget does not come from the config-applied value.
F2 (design, MEDIUM) maps to `circling-continue-hits-shed-block`: the claim that a circling continue never hits Shed's block holds only when the seam reads `ok` true.
F3 (consistency, LOW) maps to `d8-bound-reader-undecided`: D8's Bound presents visibility as a bound while no reviewer checks marked entries.
All three keys recur from rounds 2 and 3, so they stay open; the earlier resolved keys stay resolved.
