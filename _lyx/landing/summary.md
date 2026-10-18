# Remove the per-package test budget

The per-package test-count budget is gone.
It forced no reviewed decision: the guard's failure message names the value to set a row to, so an agent raised it and moved on, and no reviewer or stencil judged the raise.
It also serialized plans, since every card adding tests had to list `cmd/lyx/testdata/test-budget.yaml` under `Edit:`, making it a false scheduling dependency between otherwise independent cards.
What keeps test count down is `PATTERN-test-economy`, enforced in review, and that stays unchanged.

## Changes

- Delete `cmd/lyx/testbudget_test.go` and `cmd/lyx/testdata/test-budget.yaml`.
- Delete the `## Test budget` section from `pattern/PATTERN-test-speed.md` and from `CLAUDE.md`.
- `contracts/stencils/loom/loom-template-alignment.md`: the shared-bookkeeping sentence no longer names the budget file; the module-table line and the PATTERN index line remain.
- `internal/burlerengine/template.yaml`: the `test-economy` lens drops its "unbudgeted test" check and keeps the other two (a top-level test that should be a row, and a row that repeats existing coverage).

## Design decisions

- No replacement test is added: a package's top-level test count can now grow with no test failing.
  Review under `PATTERN-test-economy` and the recurring audit below are the only checks on it.
- Growth is watched by a recurring test-time audit, kept as the board note `test-time-audit-recurring` rather than as code.
  It re-runs `cmd/testtiming` against `docs/benchmarks/test-suite-timing.md`, in the manner of `glyph-audit-recurring`.
- Files under `docs/research/` and `docs/benchmarks/` that mention the budget are left as written, since they record past runs.

## To check by hand

- Run `grep -rn "test-budget.yaml\|testbudget_test" .` and confirm only `docs/research/` and `docs/benchmarks/` files match.
- Run `lyx gate test ./cmd/lyx` and `lyx gate test ./internal/burlerengine`; they were not run while writing this description.
