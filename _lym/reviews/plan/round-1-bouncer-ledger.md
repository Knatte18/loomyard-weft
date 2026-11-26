---
round: 1
ledger:
  - key: done-run-clear-drops-unread-level-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failed-readchild-seen-times-unspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-land-notice-copy-silences-landed-lines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: level-unread-skip-also-skips-land-notice-copy
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: checklocalvalues-test-duplicates-runsettings-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: gate-hold-process-cap-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card3-docgo-rewrite-under-extends
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: board-upsert-help-omits-run-watch
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: overview-doc-omits-new-dobby-mail
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: nested-gate-marker-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-state-write-loses-landed-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: failing-loadconfig-row-false-reason
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loadconfig-doc-omits-range-refusals
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The gating weight sits in the run watch engine card: its done-run clear, its memory across failed or missed reads, and its copy of the `land_notice` reading.
The rest are plan-text corrections in the setting, wiring, marker and gate-hold cards, plus one cap on the gate-hold line that the decision record does not make.
