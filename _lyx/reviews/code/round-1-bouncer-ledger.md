---
round: 1
ledger:
  - key: live-fixer-resume-notice-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: notifyparent-handoff-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: orch-stencil-outside-scope
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: notice-opening-duplicated-in-stencils
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings (F1 to F4), in order: the missing live-fixer resume test, the untested `NotifyParent` hand-off, the orch stencil bullet outside the recorded scope, and the duplicated notice opening in two webster stencils.
None is gating, so the loop converged on its first round.
