---
round: 1
ledger:
  - key: unconditional-push-bound-and-config-sync
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: blocking-lock-bound-contradiction
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: both-sides-failure-reporting
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: fabric-push-path-and-rebase-recovery
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: refusal-pattern-missing-from-constraints
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: pattern-entry-no-prime-exemption
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: duplicated-assumption-and-call-site-list
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings, none carried over, since there was no prior ledger.
The gating one is the lock-wait bound: Decision 3 claims a one-push wait while Open risks admits an unbounded stall.
Three MEDIUM findings concern unstated bounds or dispositions (unconditional pushes in the prime, both-sides failure reporting, the `lyx fabric push` path and its lost rebase recovery).
The remaining three are a missing Constraints entry, a PATTERN entry lacking the prime exemption, and a duplicated assumption plus a perishable call-site list.
