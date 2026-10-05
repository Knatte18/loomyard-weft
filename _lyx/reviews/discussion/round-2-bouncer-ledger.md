---
round: 2
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
    rounds: [2]
    status: open
  - key: d4-budget-test-off-by-one
    rounds: [2]
    status: open
  - key: circling-continue-hits-shed-block
    rounds: [2]
    status: open
  - key: repeated-continue-no-ceiling
    rounds: [2]
    status: open
  - key: d7-accept-bound-loose
    rounds: [2]
    status: open
  - key: pinned-failing-smoke-tally
    rounds: [2]
    status: open
---

Round 1's seven findings are not re-raised by the fresh round-2 review, so they are marked resolved.
Round 2 raised six new findings: F1 `d8-bound-reader-undecided`, F2 `d4-budget-test-off-by-one`, F3 `circling-continue-hits-shed-block`, F4 `repeated-continue-no-ceiling`, F5 `d7-accept-bound-loose`, F6 `pinned-failing-smoke-tally`.
Gating: F1, F2 and F3, all `design` at MEDIUM.
F2 and F3 share the exact budget comparison as their root.
