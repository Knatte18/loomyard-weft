---
round: 1
ledger:
  - key: weekly-aggregates-omit-repairs-and-llm-steps
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: review-archive-siblings-counted-as-segments
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: test-and-fake-changes-as-file-paths
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: finding-branches-without-reaching-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card-15-guards-without-reaching-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: exact-join-walk-cannot-stop-at-first-trailer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: untagged-time-join-undefined-for-never-tagged-slug
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: default-since-assertion-wall-clock
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 is the first round; there is no prior ledger.
Two MEDIUM scope findings (weekly sums, archived review-segment siblings) and six LOW findings were raised.
None is gating, so the loop converged on round 1.
