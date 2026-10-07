---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
  - key: single-failed-side-names-no-side
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: fabric-push-wait-bound-undecided
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: commit-records-divergence-way-forward
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: shared-primitive-never-introduced
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's seven findings did not recur in round 2 and are carried as resolved.
Round 2 raised five new ones.
The two gating findings are design gaps: Decision 3's wait bound covers only lock waiters while a hung status-push holder blocks detached children without limit (F1), and a single failed side returns a bare sentinel naming no side (F2).
Two decision findings leave `lyx fabric push` normal mode's wait behavior (F3) and the `commit-records` way forward for a real divergence (F4) unchosen.
The last is a NIT about an unnamed "shared primitive" (F5).
