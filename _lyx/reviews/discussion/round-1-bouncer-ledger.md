---
round: 1
ledger:
  - key: merge-in-and-moved-parent-check-read-different-refs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: parent-lock-held-through-pr-rework
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: finalize-retry-merges-unchecked-parent
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: release-semantics-after-pr-gate-resume-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: land-row-lock-answer-only-in-support-log
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: review-cut-short-by-reroute-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: reroute-cap-span-and-reset-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: darn-no-merge-perch-stated-three-times
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 opened eight findings and resolved none.
The two BLOCKING design findings concern the lock and ref model: Merge-In and the moved-parent check must read the same parent ref, and the parent lock must be released on PR-Gate's edge to PR-Rework.
The two MEDIUM decision findings concern lock bounds: Finalize's merge-in-required retry, and release semantics after a PR-Gate resume.
The rest are a scope gap on batten's Land row, an unreachable reroute case in D6, the reroute cap's span, and a redundant Scope Out bullet.
