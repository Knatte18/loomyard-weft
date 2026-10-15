# Plan conflict class, conflict record and config editor guards

Batch 01 (`01-plan-conflict-class`) is the only batch, and it recorded `done` at head `e6315b91caf3887623ada7258491ad0406ade4df`.
The fork committed all six cards: the `PlanConflictError` class, the conflict record, the Webster row's reroute edge, the adapter's reroute and repeat-halt, the `EditPath` empty-result abort and the terminal guard on bare `lyx config` and `lyx config menu`.

The digest listed these deviations, files changed outside the batch's own targets, which are informational:

- `internal/loomshed/webster_test.go`
- `internal/shedadapters/webster_test.go`
- `internal/webstercli/refusalnote_test.go`
- `internal/websterengine/conflictrecord.go`
- `internal/websterengine/runlevel_test.go`

The fork reported these departures from the card text:

- The recover-batch fingerprint-refusal test row spawns a recovery, lands its report, edits the card file and calls `recover-batch` again, because a terminal-failed batch otherwise spawns a recovery instead of refusing.
- `runVerbs` in `refusalnote_test.go` gained a `*testing.T` parameter to set a temp scratch dir.
- The two `loomshed` webster producer tests that reach the stuck arm set `Geom.ScratchDir`.
- The new "plan conflict rerouted" row in `contracts/specs/refusal-spec.md` uses the class `policy guard`.

`record-batch` returned no warnings and no friction note was written.
