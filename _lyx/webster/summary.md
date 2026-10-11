# small-fixes-oct11: tokencount, loom seeding routes, rubric pointer, batten notices

Both batches recorded `done`, with no reported deviations from the declared file-ops; the only warnings were informational scope notes on test symbols touched outside a batch's own target glyphs.

## Batch 01 (cards 1–3), head `bdd9ebdf42aaf5bf3f46cd33044392683d57803c`

- Card 1: tokencount counts a gap into a turn-start line, or over the 30-minute stall cap, as idle.
- Card 2: a reused slug counts only transcripts that start after the slug's previous run.
- Card 3: `lyx loom step` and `lyx loom run` help name `lyx loom start` and the driverless `lyx shed seed`.
  It adds one help test and raises the `internal/loomcli` row in `cmd/lyx/testdata/test-budget.yaml`.
- `TestCountOrch` expectations changed because its 1–2 hour gaps now count as idle.

## Batch 04 (cards 4–7), head `56f77eea626fb9275fded4206cdc6cfe8b30fea4`

- Card 4: the missing-seed refusal for run and step, and run's status-file refusal, name the driverless seed route, with refusal-spec rows and test coverage.
- Card 5: the disagreeing-seed refusal names `rm` of the seed and a re-seed with the recorded parent (`disagreeingSeedRefusal`), with a spec row and a test.
- Card 6: the plan review rubric's Proposed fixes section names `redundant-package-target` and `redundant-file-target`, with two claims in `claims_test.go`.
- Card 7: quiet, api-error and driver-dead batten notices lead with the child's state and row, and `doc.go` follows.
- The `tmux` subtest added to `TestSmokeStepBootstrapWiring` was only compiled by the fork, not run; the tmux-tier gates cover it.

The plan-level verify is left to the gate.
