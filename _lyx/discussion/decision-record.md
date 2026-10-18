# Decision record — test-budget-remove

## Goal

Remove the per-package test-count budget, which forces no reviewed decision and serializes plans through one shared file, and watch suite growth instead through a recurring test-time audit kept on the board.

## Scope

- Delete `cmd/lyx/testbudget_test.go` and `cmd/lyx/testdata/test-budget.yaml`.
- Delete the `## Test budget` section of `pattern/PATTERN-test-speed.md`.
- Delete the `## Test budget` section of `CLAUDE.md`.
- Drop the budget file from the alignment stencil's shared-bookkeeping list in `contracts/stencils/loom/loom-template-alignment.md`.
- Drop the budget clause from the `test-economy` lens in `internal/burlerengine/template.yaml`.
- Add a board note for a recurring test-time audit, measured with `cmd/testtiming` against `docs/benchmarks/test-suite-timing.md`.
- Out of scope: `docs/research/` and `docs/benchmarks/` files that mention the budget; they record past runs and stay as written.

## Decisions

### D1: Delete the guard outright, with no replacement test

Decision: remove the guard test and its data file, and add no other count or wall-time check in their place.
Rationale: the operator decided the budget has no purpose; `PATTERN-test-economy`, enforced in review, is what keeps test count down.
Bound: a package's top-level test count may now grow with no test failing; review under `PATTERN-test-economy` and the recurring audit are the only checks on it.

### D2: The recurring audit is a board note, not code

Decision: create a `kind: note` board entry `test-time-audit-recurring` describing a periodic `cmd/testtiming` run compared against `docs/benchmarks/test-suite-timing.md`, modelled on `glyph-audit-recurring`.
Rationale: the board entry asks for the audit "on the board"; it is a reminder to the operator, and writing another board entry is allowed under the run's board lock.

### D3: The alignment stencil keeps its bookkeeping rule for the remaining files

Decision: the shared-bookkeeping sentence keeps the module-table line and the PATTERN index line, and only the budget file is dropped.
Rationale: those two remain shared files that two runs touch without a design conflict.

### D4: The test-economy lens keeps its two remaining checks

Decision: the lens still reports a new top-level test that should be a row and a row that repeats existing coverage; only the unbudgeted-test clause goes.
Rationale: those checks are `PATTERN-test-economy` itself, which stays.

## Constraints

- `PATTERN-test-economy` stays unchanged and becomes the sole guard on test count.
- Markdown edits keep semantic line breaks.
- No change to the running task's own board entry.

## Auto-mode assumptions

- `docs/benchmarks/test-suite-timing.md` stays as written, since it is a measurement report of a past state.
- The `PATTERN.md` index line for `PATTERN-test-speed` needs no edit, since it does not mention the budget.

## Open risks

- A test or doc outside the found mentions may name the budget file; the full verify run catches a test that reads it.

## Acceptance criteria

- `cmd/lyx/testbudget_test.go` and `cmd/lyx/testdata/test-budget.yaml` no longer exist.
- No file outside `docs/research/` and `docs/benchmarks/` names `test-budget.yaml` or `testbudget_test.go`.
- `lyx gate test ./cmd/lyx` and `lyx gate test ./internal/burlerengine` pass.
- The board holds a note `test-time-audit-recurring` naming `cmd/testtiming` and `docs/benchmarks/test-suite-timing.md`.
