# Plan gates can name packages inside a nested fixture module, which cannot pass

`plan-gate-nested-module` · Note · bug

Imported from [issue #451](https://github.com/Knatte18/loomyard/issues/451).

## What happened

Two cards in one task had gates running `go test -tags integration` on package paths under a `testdata/` fixture directory.
The same card (or an earlier one) creates a `go.mod` in that directory, which makes the fixture its own module.
The main module then no longer contains those packages, so `go test` reports `main module does not contain package ... [setup failed]` and the gate's `&&` chain cannot pass as written.
The implementing agent worked around it by running `go vet` and `go test -tags integration ./...` from inside the fixture module, and wrote a friction note each time.

A related symptom in the same batch: the test-budget guard (`cmd/lyx/testdata/test-budget.yaml`) needed a package row raised on several cards, and that file was on no card's target list.

## Why it is abnormal

The plan was accepted through the plan gate although its gates were unsatisfiable by construction.
Plan validation does not check that a gate's package paths stay inside the module that the plan's own cards create or change.
Cards also do not predict the test-budget row they will cross, so each agent edits an untargeted file.

## What to do

- At plan-write or plan-review time, flag a gate command whose package path lies under a directory that a card makes a nested module (a `go.mod` in a card target), and require the gate to run inside that module.
- Consider listing `cmd/lyx/testdata/test-budget.yaml` as an implicit target, or surfacing the budget row in the card template, when a card adds tests.

Provenance: task `plan-glyph-precision`; notes `01-glyphchain-plan-gate.md`, `07-redundant-package-target.md`.

