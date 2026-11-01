---
round: 1
ledger:
  - key: join-dead-runner
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: lease-shortened-by-later-front
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: median-abandoned-call-unmarked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card-verify-runner-slot-phase-cancel
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runner-verb-unbounded-direct-call
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: lease-write-before-spawn
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wait-constraint-operator-commands
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: second-front-reruns-gate
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: replay-bound-contradicts-75-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-stencils-hold-no-300000
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fork-cache-lifetime-no-evidence
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings (F1 to F11, in the order of the keys above).
F1 is the only BLOCKING: the join rule never checks that the runner is alive.
F2 to F5 are MEDIUM design findings on lease renewal, the median's abandonment mark, the card-verify runner and the hidden runner verb's bound.
The remaining findings are LOW; four are local corrections whose text the review already fixes.
