---
round: 1
ledger:
  - key: exempt-files-not-carried-by-named-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: settle-records-and-runs-root-docs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
The gating one says the exempt run-state files, the settle log and the stop markers, are not staged by the status or records commit that the comments name as their carrier.
The other is a stale-doc finding on the `SettleRecords` and `RunsRootRel` comments in `internal/shedrun/paths.go`.
