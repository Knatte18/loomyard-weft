# Webster round 3 — fixer report

Both findings were valid against the code and card 4/card 7, and both are fixed.

## Fixed

### F1 — the guid form's stop refusals are unpinned (MEDIUM, scope)

Commit `c9c93df30`.
Added `TestStopStrand_UnrecordableStopRemovesNothing` in `internal/shuttleengine/stop_test.go`, with two rows: a corrupt `run.json` (the read refusal) and a read-only run directory (the write refusal).
Each row asserts an error, no `RemoveStrand` call and a byte-identical `run.json`.
Both rows call `stopRecorded`, which is `StopStrand` after `FindRun`.
`FindRun` skips an unreadable record, so only a record that turns unreadable after the scan reaches the read refusal; the test comment says so.
I confirmed each row's error comes from its intended return: "read the run record" for the first row and "record the stop … create temp: permission denied" for the second.
The test adds a top-level test, so the same commit raises `internal/shuttleengine` in `cmd/lyx/testdata/test-budget.yaml` from 158 to 159.

### F2 — the first Enter can go out after the submit window closed (LOW, design)

Commit `c2dfa4243`.
I took the code fix, not the Bound widening.
`awaitSettledBox` in `internal/shuttleengine/send.go` now checks the window again when two reads agree.
If two reads agree only at or after `closeAt`, it fails with `ErrSubmissionNotLanded` ("the input box settled only after the … submit window closed; no Enter was sent"), and `failUnlanded` then clears the send's own text as usual.
Added the row "box that settles only after the window closes fails with no Enter" to `TestRun_Send_ConfirmsSubmission`.
That row was a coverage gap, and I watched it fail on the pre-fix `send.go`.
The doc comment on `awaitSettledBox` and the matching sentence in `internal/shuttleengine/doc.go` now state the rule.

## Verification

- `go test ./...`: green.
- `go test -tags integration` over `internal/shuttleengine/...` and `cmd/...`: green.
- `go vet` (untagged and `integration`) over `internal/shuttleengine/...`: clean.
- `lyx loom lint-comments`: ok.

`gofmt -l` lists `internal/shuttleengine/run_test.go` (alignment in two table rows from earlier rounds) and `internal/shuttleengine/waiting_test.go`.
Neither is part of a finding, so I left both alone.

## Disputed

None.

## Deferred

None.
