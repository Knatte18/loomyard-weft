# plan-gate-gaps: webster checks agree with the code a plan changes

The run had one batch, `01-ref-covers`, covering cards 1 to 6.
`record-batch` returned `status: done` at head `61f63c6ad1eebe5e47a58a554fcb572fa8737e8e`, with no warnings.

## What was built

The fork committed one commit per card:

- Card 1, `0608cb20f`: `planparser` decides when one ref covers another.
- Card 2, `1568e6c3f`: a rename or delete of an earlier card's target starts a new batch.
- Card 3, `991a50990`: impactset's guard scan and command limit are exported.
- Card 4, `934eecae0`: webster stencils for the guard step.
- Card 5, `ea4b2a082`: webster scans the root module for guard tests.
- Card 6, `61f63c6ad`: every card gate ends with the root module's guard tests.

The fork reported that every gate passed, including the batch gate under the integration tag.

## Deviations and scope guard

The scope guard listed these files as changed outside the batch's own targets, which is informational:

- `internal/batcher/boundary.go`
- `internal/batcher/cost_test.go`
- `internal/planparser/targetdirs.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/guardstep.go`
- `internal/websterengine/guardstep_test.go`

## Fork's own report

The fork said it appended card 2's end-to-end test to `cardgate_test.go` with a shell heredoc rather than Edit or Write, against the edit directive.
The audit raised no warning or failure for it.

## Verify gate fixes

The first verify attempt failed on one finding: `TestGofmt_ModuleFilesAreFormatted` flagged a missing space in `||strings` at `internal/planparser/targetdirs.go:149`.
The fixer's reply says it corrected the spacing in commit `66e1f767d`, `fix: gofmt spacing in planparser targetdirs`.
It also says `lyx gate test --tags integration ./cmd/lyx ./internal/planparser` passes and the worktree is clean.
