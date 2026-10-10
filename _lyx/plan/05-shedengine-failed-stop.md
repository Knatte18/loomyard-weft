# Card 5 — shedengine-failed-stop

**Create:**
- `internal/shedengine#FailedStopRequest`
- `internal/shedengine#FailedStopResult`
- `internal/shedengine#WriteFailedStop`
- `internal/shedengine/failedstop_test.go`

**Edit:**
- `internal/shedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedengine#Shed.persistTransient`
- `internal/shedengine#ErrShedBusy`

**Intent:** The one engine function of D5 that leaves a run non-running after a failure stop, so `State` and `Error` stay written by Shed alone.
`FailedStopRequest` carries the told `StatusPath`, `LockPath` and `StatusLockPath`, the error text, the `TransientClass` and the way-forward clause the error text ends in.
`WriteFailedStop`, in a new `failedstop.go`, takes the run lock without waiting; when another holder has it, it writes nothing and reports `Busy`.
Holding the lock, it reads the status file strictly and writes `failed` with the error, the class and the file's history untouched, through the same mutate shape `persistTransient` uses (activity recomposed, `pause_requested` and the conditions left alone), only while the file still reads `running`; a file in any other state is left untouched and reported `NotRunning`, and a missing file is an error.
`FailedStopResult` reports `Written`, `Busy` and `NotRunning`.
No `CommitStatus` runs here: the next step's persist commits the file, as `Goto`'s write is committed.
`failedstop_test.go` holds one table-driven `TestWriteFailedStop`: a running file is written failed with history kept; a held run lock writes nothing and reports busy; a `paused` and a `blocked` file are left untouched and reported not running; a missing file is an error.
`doc.go` names the write as a Shed write under the run lock in the external-writer lock contract section.
`test-budget.yaml` raises `internal/shedengine` from 134 to 135.

**ImpactSummary:** A new package-level function with no caller yet; the two edited files are a doc comment and a budget row.
