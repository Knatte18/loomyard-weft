---
round: 1
ledger:
  - key: round-numbering-vs-burler-rounds
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: bouncer-nowhere-entries-placement
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d11-migration-without-discussion-producer
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: advisors-startable-from-local-layers-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: template-defaults-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: numeric-seat-name-collides-with-lead
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: goal2-contradicts-migrated-names
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: arbiter-reader-cannot-draw-unreadable-attempt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: attempt-without-verdict-reads-running-forever
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-pause-record-retention-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d8-guard-bound-wrong-direction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 was the first review; there is no prior ledger.
The two BLOCKING keys are the round-numbering premise in D2 and D11's migration over documents that lack `discussion_producer`.
Design-class MEDIUM keys are the unbounded advisor reach and D3's arbiter reader.
The rest are local consistency and bound statements.
