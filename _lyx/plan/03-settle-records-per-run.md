# Card 3 — settle-records-per-run

**Create:**
- `plan:internal/shedrun#SettleRecords` -> `func SettleRecords(l *lyxcwd.Location, runID string) string`
- `plan:internal/battencli#childGoVerifyRetrySpent` -> `func childGoVerifyRetrySpent(taskLocation *lyxcwd.Location) (bool, error)`
- `plan:internal/battencli#TestChildGoVerifyRetrySpent_ReadsThePerRunRecords` -> `func TestChildGoVerifyRetrySpent_ReadsThePerRunRecords(t *testing.T)`

**Delete:**
- `internal/shedrun#SettleRecordsFile`

**Edit:**
- `internal/shedrun#settleRecordsFileName`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun/doc.go`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#goDriverDeps`
- `internal/loomcli#goDriver.reportFor`
- `internal/loomcli#TestGoDriver_Settle`
- `internal/arbiterstop#HoldReading`
- `internal/arbiterstop#holdFixture.settle`
- `internal/battencli#battenCLI.wire`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Give the Go driver's settle records the per-run home D3 asks for, so a recipe without loom's friction directory (batten) keeps them, and point every reader at it (D3, D8).

- `SettleRecords` returns `settleRecordsFileName` (`go-settle.jsonl`, unchanged) under `RunDir(l, runID)`, beside the drive reports and the tackler directory, so a loom or darn run's records commit stages it with the run directory and a hand-over's dirty-path check counts it as run state.
  `SettleRecordsFile`, the friction-directory form, is deleted; no run holds records there today (D3), so no migration runs.
  The constant's doc and `doc.go`'s Go-driver paragraph name the new function.
- `TestPathConstructors` replaces its `SettleRecordsFile` row with a `SettleRecords` row whose want is the run directory joined with `go-settle.jsonl`.
- Readers: loom's driver wiring tells `shedrun.SettleRecords(location, c.runID)` as the settle path; `HoldReading` reads `shedrun.SettleRecords(location, shedrun.SelfRunID)`, and its `holdFixture.settle` writes there, so `TestHoldReading`'s hand-over rows exercise the new path.
- Batten's publish retry reads the child's records through the new `childGoVerifyRetrySpent` in `internal/battencli/publishretry.go`, which `battenCLI.wire`'s `GoVerifyRetrySpent` closure calls after locating the task worktree; it reads `shedrun.SettleRecords(taskLocation, shedrun.SelfRunID)` and returns `sheddriver.GoVerifyRetrySpent` of them.
  `TestChildGoVerifyRetrySpent_ReadsThePerRunRecords` in `publishretry_test.go`, over a `lyxcwd.Location` on a temp hub as `TestReadChildPublishFailure_MapsTheRecord` builds one, asserts: no records file reads false with no error; a verify-retry record at the per-run path reads true; the same record at the old `LoomFrictionDir` location reads false.
  battencli's row in `cmd/lyx/testdata/test-budget.yaml` rises from 79 to 80 for it.
- The stop report's "Friction directory" line was `filepath.Dir` of the settle path, which now names the run directory.
  `goDriverDeps` gains a `FrictionDir` field the report quotes instead, and `goDriverDepsFor` tells it `loomengine.LoomFrictionDir(location)`, today's value.
  A `TestGoDriver_Settle` row's `wantReportHas` asserts the report quotes the told friction directory, which a settle path in another directory would fail.

**ImpactSummary:** one shedrun path constructor replaced; its three readers (loom drive wiring, arbiterstop hold reading, batten publish retry), their fixtures and the stop report's friction line follow, with no on-disk migration.
