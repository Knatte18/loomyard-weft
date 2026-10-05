---
round: 5
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
    rounds: [2, 3, 4, 5]
    status: open
  - key: d4-budget-test-off-by-one
    rounds: [2, 3, 4]
    status: resolved
  - key: circling-continue-hits-shed-block
    rounds: [2, 3, 4]
    status: resolved
  - key: repeated-continue-no-ceiling
    rounds: [2]
    status: resolved
  - key: d7-accept-bound-loose
    rounds: [2]
    status: resolved
  - key: pinned-failing-smoke-tally
    rounds: [2]
    status: resolved
  - key: d4-budget-test-stated-repeatedly
    rounds: [5]
    status: open
  - key: round-entry-duplicates-d4-d8
    rounds: [5]
    status: open
  - key: residual-halt-no-acceptance-criterion
    rounds: [5]
    status: open
---

Round 5 raised four findings, none design-class and none BLOCKING.
The review confirms D4's budget test and the circling-continue claim agree with the code, so `d4-budget-test-off-by-one` and `circling-continue-hits-shed-block` are resolved.
F1 (MEDIUM, consistency) is `d4-budget-test-stated-repeatedly`, F2 (MEDIUM, consistency) is `round-entry-duplicates-d4-d8`, F4 (LOW, scope) is `residual-halt-no-acceptance-criterion`.
F3 (LOW, consistency) overclaims "attributable" and "bounded by judgment", so `d8-bound-reader-undecided` stays open as a LOW wording defect.
