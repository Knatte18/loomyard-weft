---
round: 1
ledger:
  - key: d5-marker-path-stale-or-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-resume-partial-or-mixed-round
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-d6-d7-join-gate-ordering-single-gate
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d8-disputed-criterion-presumes-checker
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-resolution-at-config-load
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: fixer-deadline-vs-waiting-time
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: auto-mode-assumptions-restate-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review, so there is no prior ledger.
Eight findings opened: F1 maps to `d5-marker-path-stale-or-unreachable`, F2 to `d6-resume-partial-or-mixed-round`, F3 to `d1-d6-d7-join-gate-ordering-single-gate`, F4 to `d8-disputed-criterion-presumes-checker`, F5 to `d9-resolution-at-config-load`, F6 to `decisions-without-acceptance-criterion`, F7 to `fixer-deadline-vs-waiting-time`, F8 to `auto-mode-assumptions-restate-rationale`.
The gating ones are the two MEDIUM design entries, the marker path and the Disputed criterion.
