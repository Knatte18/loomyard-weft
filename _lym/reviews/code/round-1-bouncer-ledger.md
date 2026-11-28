---
round: 1
ledger:
  - key: dotnet-redfirst-green-row-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dotnet-redfirst-selected-tests-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dotnet-redfirst-row-time-claim-unmeasured
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all on `internal/gatecli/redfirstdotnet_integration_test.go`.

- `dotnet-redfirst-green-row-dropped` (F1): no real `dotnet test` run in red-first's exported checkout exits 0 any more,
  and the `Loosened-Check:` cause quotes a serial cost although the rows now run in parallel.
- `dotnet-redfirst-selected-tests-unasserted` (F2): the red row would pass if the passing test were never selected.
- `dotnet-redfirst-row-time-claim-unmeasured` (F3): the doc comment states where a row's time goes without a breakdown.
