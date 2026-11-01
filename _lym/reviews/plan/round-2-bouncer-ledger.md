---
round: 2
ledger:
  - key: cancel-end-only-reads-done
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-warning-way-forward-not-fabric-pull
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: fix-task-gate-label-unknown-vocabulary
    rounds: [1]
    status: resolved
  - key: adopt-resume-texts-unlisted
    rounds: [1]
    status: resolved
  - key: gateslot-doc-table-subjectless-check
    rounds: [1]
    status: resolved
  - key: stale-file-headers-check-adopt-deps
    rounds: [1]
    status: resolved
  - key: settle-ownership-drop-race-only
    rounds: [1]
    status: resolved
  - key: failedend-run-status-before-record
    rounds: [1]
    status: resolved
  - key: live-pair-row-hits-takeover-stop
    rounds: [1]
    status: resolved
  - key: card5-missing-resign-arrows
    rounds: [1]
    status: resolved
  - key: filefronttask-refusal-unnamed
    rounds: [1]
    status: resolved
  - key: failed-tier-notice-keeps-prime-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: overview-batten-paragraph-adopt-first-pass
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings; round 2 raised four.
Ten round-1 keys do not reappear and are resolved.
`cancel-end-only-reads-done` returns as F1: the end-only error exists, but no named test reaches the `Cancel` call that produces it, so the finding stays open as a design MEDIUM.
`d2-warning-way-forward-not-fabric-pull` returns as F4, now a scope finding about warning on a branch the reconcile moved.
F2 (the failed-tier notice's hand way forward in the prime) and F3 (the overview's batten paragraph) are new.
