---
round: 1
ledger:
  - key: d8-board-decision-vs-run-lock
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: in-flight-peer-set-inconsistent
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d5-bound-overstates-agent-landing
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-forgery-not-compared-with-today
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d4-post-fallback-verdict-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d7-false-arbiter-precedent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: missing-acceptance-scenarios-d6-d7
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: d8-misses-empty-deploy-cue
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: q4-uncertainty-contradicts-d3
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: d6-prime-form-no-caller-run
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings, all open.
The blocker is D8: an operator decision on a running task cannot be written into its board body under the run lock, and D5's auto-landing bound relies on that rule.
The peer-set definition (D3/D6) and D5's Bound wording are the MEDIUM items; the rest are small consistency, scope and acceptance gaps.
