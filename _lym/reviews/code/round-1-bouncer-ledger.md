---
round: 1
ledger:
  - key: readme-d2-clone-description-missing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: add-anchor-does-not-move-forked-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-fsm-background-protected-contradicts-d7
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: battencli-one-repo-seam-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-over-parked-merge-pinned-for-mergein-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: replay-nonreplayed-warp-sha-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: origin-reconcile-one-repo-code-stop-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: integrateweft-one-repo-stop-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: one-repo-teardown-error-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: index-lock-retry-no-real-path-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-merge-record-update-leaves-warpstart-stale
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: judge-does-not-clear-pushowedwayforward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-thresholds-assertion-narrowed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
It raised 13 findings: one BLOCKING (the README lacks the D2 clone description and hazards), five design-class MEDIUMs and one consistency MEDIUM, and six LOW.
The two design-wrong entries change behavior: the one-repository `--anchor` Add must move the forked records, and the merge-record update must not leave `WarpStart` stale after HEAD moves.
The rest are documentation corrections or missing test rows whose text the findings already fix.
