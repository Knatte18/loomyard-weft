---
round: 1
ledger:
  - key: status-oneshot-test-text-vs-steady
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: checklist-poll-text-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: status-watch-unavailable-key-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW test-coverage findings and no others.
F1 maps to `status-oneshot-test-text-vs-steady`, F2 to `checklist-poll-text-untested`, F3 to `status-watch-unavailable-key-unpinned`.
None is gating, so the loop converges.
