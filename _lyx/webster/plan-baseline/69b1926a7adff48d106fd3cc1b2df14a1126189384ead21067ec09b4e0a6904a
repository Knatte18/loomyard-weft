# Card 2 — sheddriver-records

**Create:**
- `internal/sheddriver/doc.go`
- `internal/sheddriver/testmain_test.go`
- `internal/sheddriver#StopKey`
- `internal/sheddriver#StopKeyOf`
- `internal/sheddriver#writeRecordJSON`
- `internal/sheddriver#readRecordJSON`

**Move:**
- `internal/loomengine/settle.go`
- `internal/loomengine/settle_test.go`
- `internal/loomengine/tackler.go`
- `internal/loomengine/tackler_test.go`
- `internal/loomengine#TestStopKeyOf`

**Delete:**
- `internal/loomengine#ArbiterStopKey`
- `internal/loomengine#StopKeyOf`

**Edit:**
- `internal/loomengine#ArbiterAttempt`
- `internal/loomengine#TestArbiterAttempts`
- `internal/arbiterstop/holdreading.go`
- `internal/arbiterstop/holdreading_test.go`
- `internal/battencli#readTacklerAsArbiterMarker`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestArbiterInFlightFrom`
- `internal/loomcli/arbiter.go`
- `internal/loomcli/arbiterconsume.go`
- `internal/loomcli/drive.go`
- `internal/loomcli/drivehandback.go`
- `internal/loomcli/driverules.go`
- `internal/loomcli/drivewiring.go`
- `internal/loomcli/tackler.go`
- `internal/loomcli/tacklerconsume.go`
- `internal/loomcli/tacklerprompt.go`
- `internal/loomcli/arbiter_test.go`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `internal/loomcli#TestRenderChecklist`
- `internal/loomcli/drive_test.go`
- `internal/loomcli/drivehandback_test.go`
- `internal/loomcli/driverules_test.go`
- `internal/loomcli/tackler_test.go`
- `internal/loomcli/tacklerconsume_test.go`
- `internal/loomcli/tacklerprompt_test.go`
- `internal/reportengine#arbiterAt`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Open `internal/sheddriver`, the shed-layer package D1 names, with the records both the Go driver and loom's arbiter read: the stop key, the settle records and the tackler's records.
Nothing about how they are written, read or named on disk changes, so the moved tests are the coverage.

- `doc.go` opens the package doc: the shed layer's Go driver and tackler, a told-geometry engine that is handed every path it works on and gets recipe behaviour through seams the arming module fills.
  This card's doc covers the records; later cards extend it as the driver, tackler and config land.
  `testmain_test.go` runs the package's tests under `tmuxkit.Main`, as `internal/loomengine/testmain_test.go` does.
- `StopKey` replaces `loomengine.ArbiterStopKey` with the same three fields and JSON keys (`current_producer`, `history_length`, `state`), so every attempt and settle record already on disk reads unchanged; `StopKeyOf` replaces `loomengine.StopKeyOf`, and both live in `internal/sheddriver/stopkey.go`.
  `TestStopKeyOf` moves to `internal/sheddriver/stopkey_test.go`.
- `internal/loomengine/settle.go` and its test move to `internal/sheddriver/settle.go` and `settle_test.go`; `internal/loomengine/tackler.go` and its test move to `internal/sheddriver/tacklerrecords.go` and `tacklerrecords_test.go`, since the tackler call takes `tackler.go` in card 4.
  Exported names stay as they are (`SettleRecord`, `AppendSettleRecord`, `ReadSettleRecords`, `GoVerifyRetrySpent`, the `Settler*` and `Settle*` words, `TacklerRole`, the three verdict words, `WorktreeSnapshot`, `TacklerAttempt`, `TacklerVerdict`, `TacklerEnvelope`, `TacklerMarker` and their readers and writers).
  Their `ArbiterStopKey` fields become `StopKey`, error prefixes `loom:` become `sheddriver:`, and the two file headers drop the "sit in this package, not in loomcli" rationale for the package doc's.
- The tackler records wrote and read through loomengine's `writeArbiterJSON` and `readArbiterJSON`; `sheddriver` gets its own `writeRecordJSON` and `readRecordJSON` with the same behaviour (indented JSON plus a newline, the directory created, an absent file read as found false), since `sheddriver` may not import `loomengine`.
- `loomengine.ArbiterAttempt.StopKey` becomes a `sheddriver.StopKey`, so `loomengine` imports `sheddriver`; the reverse import never exists (D1).

Every other reference retargets: `internal/arbiterstop`'s hold reading, battencli's tackler marker read, its publish-retry settle read inside `wire` and its marker test, every listed loomcli file, and reportengine's `arbiterAt` fixture.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/sheddriver: 3` for the three moved tests.

**ImpactSummary:** two loomengine records files and the stop key change package; every reader in loomengine, arbiterstop, battencli, loomcli and one reportengine fixture retargets, with on-disk JSON unchanged.
