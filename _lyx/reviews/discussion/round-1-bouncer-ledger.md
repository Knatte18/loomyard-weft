---
round: 1
ledger:
  - key: removal-edges-unbounded-condition3
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-guard-already-exists
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shared-window-pane-moved-in-session
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: withdraw-refusal-keyed-on-phase
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: marker-removals-escape-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pause-expires-held-request
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: missing-launch-time-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tmux-empty-answer-unverified
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unnamed-false-live-cases
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: landing-conflict-stop-skips-shell-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d6-no-reclaim-inside-grace
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-skip-misses-post-resolve-teardown
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: acceptance-omits-windows-skip
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: child-shell-risk-lacks-evidence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: departure-restates-rejected-alternative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first ledger.
Keys map to F1 through F15 in order.
The BLOCKING key and the five gating MEDIUM design keys need design changes and a fresh review.
The `decision` MEDIUM (withdraw refusal) and the LOW and NIT keys are local corrections, except the D7 skip extension, which changes a decision.
