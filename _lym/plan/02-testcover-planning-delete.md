# Card 2 — testcover-planning-delete

**Delete:**
- `internal/testcover/upkeep.go`
- `internal/testcover/upkeep_test.go`
- `internal/testcover/testselect.go`
- `internal/testcover/testselect_test.go`
- `internal/testcover/declcompare.go`
- `internal/testcover/miss_string_test.go`
- `internal/testcover#Map.FailedAt`
- `internal/testcover#stampDifferences`
- `internal/testcover#Toolchain.GoRoot`
- `internal/testcover_test#TestMapFailedAt`

**Edit:**
- `internal/testcover/doc.go`
- `cmd/lym#testOnlyCodeAllowed`

**Intent:** Remove the `internal/testcover` code that only the retired map planning reached, per `PATTERN-test-only-code` (the Scope's testcover bullet, D9).
Card 1 removed its last callers: `testMapRun.planLockedPackage`, `testMapRun.headSelector` and `testMapRun.headTree` called the selector, `testcover.PlanUpkeep` and `Toolchain.GoRoot`, and the planning and audit called `Map.FailedAt`.
The files go whole: `upkeep.go` (`PlanUpkeep`, `UpkeepInput`, `UpkeepTree`, `UpkeepPlan`, the `Upkeep*` verdicts, `PlanCounts`), `testselect.go` (`Selector`, `NewSelector`, `SelectorConfig`, `Target`, `Selection`, `CommitReader`, `Miss`, `MissReason` and the `Miss*` reasons) and `declcompare.go`, which only the selector reaches.
Their tests go with them: `upkeep_test.go`, `testselect_test.go` with its fixtures, and `miss_string_test.go`, whose `Miss.String` only the selector tests call.
`coveragemap.go` loses `Map.FailedAt` and `stampDifferences`, which only `Selector.environmentMiss` called; `coveragemap_test.go` loses `TestMapFailedAt` and keeps `TestWriteMapReadMap`.
`runner.go` loses `Toolchain.GoRoot`, which no test covers.
`NestedPackageDirs`, `StampModuleFiles`, `MapTest.Measured` and the map I/O stay: the map run and `lym gate redundant` still call them.
`internal/testcover/doc.go` drops the `PlanUpkeep` and `Selector` paragraphs, the `Map.FailedAt` sentence and the `ProfiledFuncs` sentence of the function card 1 deleted, and its `GoEnvironment` line drops "and GoRoot its root directory".
`cmd/lym#testOnlyCodeAllowed` loses the entries card 1 added for these functions, since an entry that matches no unreachable function fails the guard.
No behavior changes; the remaining tests of the package cover what stays.

**ImpactSummary:** Testcover-local deletes with no production caller left after card 1; outside testcover only `cmd/lym`'s test-only-code allowlist names the removed symbols and loses those entries, and the map format and its readers are untouched.
