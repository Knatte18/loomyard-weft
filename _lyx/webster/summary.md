# lyx report timing: durfmt, caller label and timing report

All three batches reached `done`.

## 01-durfmt (cards 1–3)

The first `record-batch` failed the done-check: the plan's Create targets `TestParse` in `internal/sleepjournal` and `TestParseSettleRecords` in `internal/shedrun` did not resolve.
One `recover-batch` completed with `status: done`, head `b66a229c9`.
The pane duration formatters now live in `internal/durfmt`, the journal sleep parser in `internal/sleepjournal`, and the settle-record format and decoder in `internal/shedrun`.
Digest deviations, informational: `internal/arbiterstop/holdreading.go`, `internal/arbiterstop/holdreading_test.go`, `internal/battencli/drive.go`, `internal/battencli/publishretry_test.go`, `internal/durfmt/testmain_test.go`, `internal/loomcli/drivewiring.go`, `internal/sheddriver/drive.go`, `internal/sheddriver/driverules.go`, `internal/sleepjournal/sleepjournal.go`, `internal/sleepjournal/testmain_test.go`, `tools/tokencount/sleep.go`.

## 04-caller-label (cards 4–8)

Recorded `done` at head `611a64d36`.
It adds the caller label classification, the `By` and `PendingBy` history fields, the label wiring through the goto and step verbs, the Go driver and batten, `gitrepo.BranchRefs` with `Branches` and `SnapshotAt` on `reportengine.Sources`, and `ComputeTiming`.
Digest deviations, informational: `internal/reportengine/fake_test.go`, `internal/reportengine/timing.go`, `internal/reportengine/timing_test.go`, `internal/reportengine/timingspans.go`, `internal/shedengine/goto.go`, `internal/shedengine/goto_test.go`, `internal/shedengine/run.go`.

## 09-timing-waits-attempts (cards 9–12)

Recorded `done` at head `bf44d0716`.
It adds waits and arbiter and alignment attempt time, stops by settler, the markdown renderer and the `lyx report timing` verb.
Digest deviations, informational: `internal/reportengine/batten.go`, `internal/reportengine/timing.go`, `internal/reportengine/timing_test.go`, `internal/reportengine/timingmarkdown.go`, `internal/reportengine/timingspans.go`, `internal/reportengine/timingstops.go`, `internal/reportengine/timingwaits.go`.

## Scope guard

No digest carried warnings, and every deviation list was informational.
