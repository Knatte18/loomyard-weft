---
round: 1
ledger:
  - key: cancel-end-only-reads-done
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-warning-way-forward-not-fabric-pull
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fix-task-gate-label-unknown-vocabulary
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-resume-texts-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gateslot-doc-table-subjectless-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-file-headers-check-adopt-deps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: settle-ownership-drop-race-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: failedend-run-status-before-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: live-pair-row-hits-takeover-stop
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-missing-resign-arrows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: filefronttask-refusal-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings and no ledger preceded it.
The one gating finding is the end-only cancel in card 4 that card 6's `Remove` counts as done, against D8.
The `gate` label question (F3) needs the parent's decision before card 13 can carry it.
The rest are doc and header coverage gaps, a D2 wording departure, and two LOW guard-reach gaps.
