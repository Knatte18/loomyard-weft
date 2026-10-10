---
round: 1
ledger:
  - key: guard-never-lifted-when-turn-start-alone-in-tick
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: same-batch-position-test-not-conditioned-on-armed-guard
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: held-wait-not-ended-by-same-batch-turn-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-test-names-nonexistent-step-hook
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: observeclocks-first-observation-stamp-unrecorded
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card2-grep-misreads-routing-bounces-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, all open.
The gating ones concern the turn-start guard in cards 5 and 6: it is never lifted when the turn start arrives alone in a tick, the position test is not conditioned on an armed guard, and a same-batch turn start never ends the hold.
The rest are consistency defects in cards 4, 3 and 2.
