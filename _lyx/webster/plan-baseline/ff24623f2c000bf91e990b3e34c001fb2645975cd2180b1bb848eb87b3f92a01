# Card 8 — shedfake-seat-runner

**Create:**
- `plan:internal/testkit/shedfake#SeatRunner` -> `type SeatRunner struct`
- `plan:internal/testkit/shedfake#SeatRunner.Probe` -> `func (f *SeatRunner) Probe(table seatengine.Table) (seatengine.LiveTable, error)`
- `plan:internal/testkit/shedfake#SeatRunner.Resume` -> `func (f *SeatRunner) Resume(table seatengine.Table, live seatengine.LiveTable) (seatengine.Result, error)`
- `plan:internal/testkit/shedfake#SeatRunner.Run` -> `func (f *SeatRunner) Run(table seatengine.Table) (seatengine.Result, error)`

**Edit:**
- `internal/testkit/shedfake/shedfake_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/seatengine#Table`
- `plan:internal/seatengine#LiveTable`
- `plan:internal/seatengine#Result`
- `internal/testkit/shedfake#BurlerRunner`
- `internal/testkit/shedfake#scripted`

**Intent:** Add the fake seat runner the adapter's and the recipe's tests share (D13, PATTERN-testkit), shaped like `BurlerRunner`: scripted `Probe` answers, scripted `Run` and `Resume` results and errors by call order with the last entry repeating, recorded tables and live tables per call, and optional `RunFn`, `ProbeFn` and `ResumeFn` overrides.
It imports `seatengine` alone for its types, asserts nothing, and satisfies card 9's `shedadapters.SeatRunner` structurally.
`shedfake_test.go` gains one test pinning the scripting order and the override precedence, as `TestBurlerRunner_LastEntryRepeats` does for the burler fake.
Raise the `internal/testkit/shedfake` budget row by one.

**ImpactSummary:** One new fake beside the existing ones; no existing fake or consumer changes.
