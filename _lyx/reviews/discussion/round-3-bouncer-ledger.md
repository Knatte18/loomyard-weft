---
round: 3
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
    rounds: [2, 3]
    status: open
  - key: d4-budget-test-off-by-one
    rounds: [2, 3]
    status: open
  - key: circling-continue-hits-shed-block
    rounds: [2, 3]
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

Round 1's seven findings stay resolved.
Round 2's `repeated-continue-no-ceiling`, `d7-accept-bound-loose` and `pinned-failing-smoke-tally` are not raised by the fresh round-3 review, so they are marked resolved.
Round 3 raised three findings, all `design` at MEDIUM, each reopening a round-2 key: F1 `d4-budget-test-off-by-one`, F2 `circling-continue-hits-shed-block`, F3 `d8-bound-reader-undecided`.
Gating stays at three across rounds 2 and 3, hence CIRCLING.
