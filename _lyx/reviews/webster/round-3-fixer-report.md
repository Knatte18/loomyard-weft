# Webster round 3 — fixer report

## Fixed

- **F1** (BLOCKING, `internal/loomshed/gates_test.go:462`): the done-card-edited table in `TestNewPlanGate` now passes `""` as `NewPlanGate`'s `scratchDir` argument, matching every other call in the file and the four-argument signature main brought in.
  Validated against the publish-failure log, whose only error is this call.
  Commit `709cce21e`.
  `lyx gate test ./internal/loomshed` and `lyx gate test --tags integration ./internal/loomshed` both pass, so no further compile error was hidden behind this one.

## Round gate repair

The round gate's `lyx loom lint-comments` flagged fixed-column-wrapped comment lines that the merge-in commit brought from main, in `cmd/lyx/spawnobservability_test.go`, `internal/boardengine/doc.go`, `internal/clihelp/exec.go`, `internal/clihelp/jsonhelp.go`, `internal/clihelp/jsonhelp_test.go` and `internal/reedengine/contract_integration_test.go`.
Each flagged comment is rewrapped to one sentence per line, breaking inside a sentence only at a semicolon.
The change touches comments only.
Commit `419a25df1`.
`lyx loom lint-comments` passes on the working tree and with `--base 52ae208c0^1`.

## Disputed

None.

## Deferred

None.
