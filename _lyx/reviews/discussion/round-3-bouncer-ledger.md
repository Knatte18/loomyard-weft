---
round: 3
ledger:
  - key: unconditional-push-bound-and-config-sync
    rounds: [1]
    status: resolved
  - key: blocking-lock-bound-contradiction
    rounds: [1]
    status: resolved
  - key: both-sides-failure-reporting
    rounds: [1]
    status: resolved
  - key: fabric-push-path-and-rebase-recovery
    rounds: [1]
    status: resolved
  - key: refusal-pattern-missing-from-constraints
    rounds: [1]
    status: resolved
  - key: pattern-entry-no-prime-exemption
    rounds: [1]
    status: resolved
  - key: duplicated-assumption-and-call-site-list
    rounds: [1]
    status: resolved
  - key: wait-bound-covers-waiters-only
    rounds: [2]
    status: resolved
  - key: single-failed-side-names-no-side
    rounds: [2]
    status: resolved
  - key: fabric-push-wait-bound-undecided
    rounds: [2]
    status: resolved
  - key: commit-records-divergence-way-forward
    rounds: [2]
    status: resolved
  - key: shared-primitive-never-introduced
    rounds: [2]
    status: resolved
  - key: prime-code-side-push-failure-unbounded
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: lock-wait-expiry-no-side-no-envelope
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: envelope-text-vs-vocabulary-and-discrimination-site
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: rejection-row-class-unnamed
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: denied-forms-constraint-partial-list
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: single-side-criterion-one-sided
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: decision-1-wrong-pointer-and-repeated-risk
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 findings did not recur in round 3 and are carried as resolved.
Round 3 raised seven new findings and none is design-class.
The two MEDIUM ones are a decision gap, the unstated failure of the prime's code-side push (F1), and a consistency gap, a lock-wait expiry with no side and no `commit-records` envelope (F2).
The rest are LOW wording, class-naming and criterion-coverage findings, and one NIT.
