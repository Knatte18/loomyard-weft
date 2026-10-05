# Fixer report — round 1

All three findings are fixed, one commit each, and nothing is deferred.

## Fixed

- **F1 (MEDIUM, design)** — `b2b7fd9fd`: `internal/shedverbs/steprecord_test.go`.
  Two rows join the existing table in `TestStepCmd_ShortEnvelopeOnStdoutFullEnvelopeInRecord`, with a `hooks` column and two small helpers (`awaitingRow`, `afterStepReturning`).
  `SuccessWithFrictionAndParentNotice` asserts the short success key set carries `friction` and `parent_notice`, and `RefusalWithFriction` asserts the short error key set carries `friction`.
  No new test function was added; the gap was coverage, so the table was extended.
- **F2 (LOW, consistency)** — `0816aff73`: `gofmt` spacing in `steprecord_test.go`.
- **F3 (LOW, consistency)** — `1a4d8ac74`: `internal/shedrun/paths.go`, the `ParkMarker` comment now names the full step envelope, held by the step record, as the source of `scratch_dir`.

After the fixes `go build ./...` and `go test` over `shedverbs`, `shedcli`, `loomcli` and `shedrun` pass, and `gofmt -l` no longer lists `steprecord_test.go`.

## Deferred

None.
