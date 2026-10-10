---
round: 2
ledger:
  - key: card4-tmux-build-breaks-signaltick
    rounds: [1]
    status: resolved
  - key: cards10-11-test-budget-rows
    rounds: [1]
    status: resolved
  - key: card7-claudeengine-doc-false
    rounds: [1]
    status: resolved
  - key: card8-readiness-bound-unstated
    rounds: [1]
    status: resolved
  - key: card8-readiness-test-triplication
    rounds: [1]
    status: resolved
  - key: card5-watchnames-wallclock-test
    rounds: [1]
    status: resolved
  - key: card14-benchmark-on-live-hub
    rounds: [1]
    status: resolved
  - key: cards7-13-escape-hatch-bounds
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: card4-default-timing-floor-new-test
    rounds: [1]
    status: resolved
  - key: card6-discovery-cadence-undocumented
    rounds: [1]
    status: resolved
  - key: card13-two-cmdlyx-tests
    rounds: [1]
    status: resolved
  - key: overview-opener-decision-vs-card6-tests
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card5-discover-signal-literal-pin
    rounds: [1]
    status: resolved
  - key: card11-skip-phase-end-offset
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-resume-absent-state-no-refusal
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: shuttleengine-doc-shadow-mode-stale
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-discover-wake-bound-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card2-shuttle-floor-warn-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card14-before-binary-from-prod
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised 13 findings, 2 BLOCKING.
Round 2 re-raised none of the round-1 BLOCKING or design findings; the keys not re-raised are resolved.
Two round-1 keys map to round-2 findings: F3 (`overview-opener-decision-vs-card6-tests`, now MEDIUM scope) and F5 (`cards7-13-escape-hatch-bounds`, now LOW consistency on card 11's pointer-skip Bound).
Six keys are new in round 2.
The one gating key is `card11-skip-phase-end-offset` (F1, design MEDIUM), first raised in round 2, so it goes to Open risks.
