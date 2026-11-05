---
round: 1
ledger:
  - key: stale-step-stderr-on-dead-loop-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: panic-recovery-narrower-than-claims
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: signal-choice-unrecorded-decision
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: trace-copy-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: no-starting-mark-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: recovery-kills-stale-process-group
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: signal-path-can-write-failed-stop
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ignored-signal-leaves-loop-alive
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: partial-line-hides-exit-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: panic-subtest-misses-kill-and-stack
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: other-loops-marks-in-span-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 found eleven findings and no earlier ledger exists.
Gating: the stale step stderr (F1) and the panic recovery scope (F2).
The signal-without-envelope choice (F3) is deferred to the parent.
