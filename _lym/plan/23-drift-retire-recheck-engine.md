# Card 23 — drift-retire-recheck-engine

**Delete:**
- `internal/driftengine/recheck.go`
- `internal/driftengine/recheck_test.go`
- `internal/driftengine#requireCompareRecorded`
- `internal/shedrecipe#driftRecheckEntry`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#TestDriftEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe/doc.go`
- `internal/driftengine/doc.go`

**Intent:** Remove the `DriftRecheck` engine and its constructor (D4, D14); the row name `Drift-Recheck` stays, now on the `DriftVerify` engine since card 16, so no rename or retirement table is needed.

`verify_test.go`'s helper `requireCompareRecorded`, whose last callers are in `recheck_test.go` since card 16 rewrote Drift-Verify's test, goes with it.
The whole of `recheck.go` (`DriftRecheck`, `NewDriftRecheck`, `back`, `acceptedTests`, `recheckOpening` and the methods) and its test file go; `registry` drops `DriftRecheck`; `TestDriftEntries` drops its Recheck rows, `TestNames` and `minimalEntries` drop the name, and both doc.go files drop the `# Drift-Recheck` text and say Drift-Recheck is the `DriftVerify` engine in a second row.

**ImpactSummary:** Deletes an engine no shipped recipe names since card 16; the registry and its three tests lose one name.
