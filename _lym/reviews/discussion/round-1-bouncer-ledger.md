---
round: 1
ledger:
  - key: watchdog-exemption-left-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: done-stamp-ignores-goto
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stop-vs-checkout-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: flag-unreadable-before-session-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-describe-bound-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: acceptance-three-skipped-rows
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d6-reroute-never-reenters
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-codereviewoff-no-reader
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: publish-guard-steer-text-false
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d7-bound-miscites-run-shed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, all open.
Five are gating design findings at MEDIUM: the watchdog fact left open, the Done-count stamp across a goto, the stop-versus-checkout order, the flag read before session start, and D4's false Describe leg.
The rest are a MEDIUM and two LOW consistency findings, a LOW design finding on the publish guard's steer text, and a NIT on D7's bound.
One interactions finding was rejected by the handler, so it has no ledger key.
