# Remove the per-package test budget

The per-package test-count budget is removed.
It was meant to make each raise a visible, reviewed decision, but no role judged a raise: the alignment stencil treated the file as bookkeeping, the code reviewers were not asked, the plan review's test-economy lens only checked that a new test was budgeted, and the guard's failure named the value to set.
Its only lasting effect was a shared file most test-adding cards had to list under `Edit:`, which made it the largest false scheduling dependency between cards.
`PATTERN-test-economy`, enforced in review, stays the check on test count.

## Changes

- `cmd/lyx/testbudget_test.go` and `cmd/lyx/testdata/test-budget.yaml` are deleted.
- `pattern/PATTERN-test-speed.md` loses its `## Test budget` section; the `PATTERN.md` index line needed no edit.
- `CLAUDE.md` loses its `## Test budget` section.
- The alignment stencil (`contracts/stencils/loom/loom-template-alignment.md`) no longer names the budget file among the shared bookkeeping files that do not escalate.
- The `test-economy` lens in `internal/burlerengine/template.yaml` keeps its two remaining checks, a new top-level test that should be a row and a row that repeats existing coverage, and drops the unbudgeted-test check.

## Design decisions

- No replacement guard: a package's top-level test count can now grow with no test failing.
  Review under `PATTERN-test-economy` and a recurring audit are the checks on it.
- The recurring audit is the board note `test-time-audit-recurring`, modelled on `glyph-audit-recurring`.
  It describes a periodic `cmd/testtiming` run compared against `docs/benchmarks/test-suite-timing.md` and is not code.
- Files under `docs/research/` and `docs/benchmarks/` that mention the budget are left as written, since they record past runs.
