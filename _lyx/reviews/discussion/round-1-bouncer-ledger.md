---
round: 1
ledger:
  - key: batten-tamper-check-removal-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: loom-migrate-rewrite-permanent-reaches-run-settings
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: darn-tackler-settings-source-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: batten-halted-row-resume-no-seam
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d2-bound-line-overstates-hand-back
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-no-go-seeded-run-premise-false
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-miss-d2-d7-behaviour
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tackler-timeout-min-zero-meaning-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: abandoned-session-named-twice
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

First round, no prior ledger.
Two BLOCKING design gaps: the batten tamper check is dropped without a stated bound, and the loom.yaml Migrate rewrite is permanent and also fires on run_settings.
Two MEDIUM consistency gaps: the darn tackler settings source and the batten halted-row-only resume lack a home in the seam list.
The remaining findings are LOW or NIT: an overstated Bound line, a false no-go-seeded-run premise, missing acceptance criteria, an open meaning for a set zero timeout, and a duplicated clause.
