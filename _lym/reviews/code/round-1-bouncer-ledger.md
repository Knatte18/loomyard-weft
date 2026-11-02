---
round: 1
ledger:
  - key: benchmark-report-revision-not-checkable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: benchmark-pair2-and-shm-figures-not-derivable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lymtemp-runnamedigits-misnamed-and-pins-tally
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: tmuxkit-widest-uid-row-retired-run-name-shape
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four consistency findings, two LOW and two NIT, none gating.
Two concern the new benchmark report (a checkable revision, and traceability of the pair 2 window and shm file figures).
Two concern the run-name change (the `runNameDigits` name and comment in `internal/lymtemp/run.go`, and a stale fixture row in `internal/testkit/tmuxkit/tmuxkit_test.go`).
