# Card 15 — No GitHub issue for a recurring finding

**Edit:**
- `internal/loomengine/anomaly.go`
- `internal/loomengine/anomalybody.go`
- `internal/loomengine/anomaly_test.go`
- `internal/loomengine/anomalybody_test.go`
- `internal/loomcli/selfreport.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/selfreport_test.go`
- `docs/overview.md`

**Delete:**
- `internal/shedadapters/ledgeraccess.go`
- `internal/shedadapters/ledgeraccess_test.go`

**Intent:**
D11: the `recurring-finding` anomaly kind, its detector trigger, its ledger-observation input and its selfreport dedup branch are removed; the escalation to the parent now covers what issue #359 duplicated.

- `loomengine/anomaly.go`: remove `AnomalyRecurringFinding`, `LedgerObservation`, `recurringFindingThreshold`, `detectRecurringFinding` and `renderRecurringFindingTitle`, and the trigger-5-only `Anomaly` fields (`Round`, `BouncerRow`, `LedgerKey`, `LedgerRounds`, `LedgerStatus`).
  `DetectAnomalies` drops its `ledgers` parameter.
  The file header and comments say four triggers.
  `bounce-budget-exhausted` stays: review segments no longer reach it, `PR-Gate` still does.
- `loomengine/anomalybody.go`: remove the recurring-finding body branch.
- `loomcli/selfreport.go`: remove `discoverLedgers`, the `IsLedgerPath`/`ReadLedger` deps fields and their doc, and the recurring-finding branch of `collapseAnomaliesByTitle`, whose doc then states that one detection returns at most one anomaly per title shape.
  `arm.go`'s `anomalyDeps` stops filling the removed fields and drops its `shedadapters` import if unused.
- `shedadapters/ledgeraccess.go` and its test exist only for that input and are deleted; grep for `IsLedgerPath`, `ReadLedger`, `LedgerEntry` and `shedadapters.Ledger` and leave no reference.
- `docs/overview.md`'s selfreport entry lists four anomalies.

Tests: `anomaly_test.go` drops the trigger-5 tables and asserts that a status whose history publishes a ledger with a key open across three rounds yields no anomaly beyond the halt kind;
`selfreport_test.go` drops the ledger-discovery cases and keeps a case where a final status with such a ledger in its history files nothing for it;
`anomalybody_test.go` drops the recurring-finding body case.

**ImpactSummary:** Removes one anomaly kind and the exported ledger accessor end to end; a marker title of the old shape stays in a machine-local marker and is never re-filed.

**Commit:** 15: No GitHub issue for a recurring ledger finding
