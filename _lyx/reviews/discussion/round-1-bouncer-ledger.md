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
    fix: mechanical
  - key: batten-resume-halted-row-only-no-seam
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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

Round 1 raised nine findings, all first raised this round.
The two BLOCKING ones are design flaws in D4 (no stated bound after removing the batten tamper snapshot) and D5 (a stateless value-equality Migrate rewrite that erases pins and fires on run_settings).
The MEDIUM pair is missing wiring statements: where a darn run's tackler keys come from, and where batten's halted-row-only resume is enforced.
The rest are a mis-stated bound, a false premise, missing acceptance criteria, an undefined set-zero meaning and a duplicated clause.
