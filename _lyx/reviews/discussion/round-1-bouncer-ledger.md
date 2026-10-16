---
round: 1
ledger:
  - key: d4-batten-tamper-check-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-migrate-rewrite-permanent-and-reaches-run-settings
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-darn-tackler-settings-source-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d4-batten-halted-row-resume-has-no-seam
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
  - key: d5-tackler-timeout-min-zero-meaning-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d4-abandoned-session-named-twice
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings on the discussion pair, all still open.
Two are BLOCKING design findings: D4 removes batten's tamper snapshot without a stated bound, and D5's Migrate rewrite is permanent and also fires on `run_settings`.
Two MEDIUM consistency findings concern where a darn run's tackler settings come from and where batten's halted-row-only resume is enforced.
The rest are LOW or NIT wording and coverage fixes.
