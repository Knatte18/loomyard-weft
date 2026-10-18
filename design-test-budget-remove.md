# Remove the per-package test budget; watch suite growth by a recurring test-time audit

`test-budget-remove` · Task · enhancement, loom, burler, stencil · done

## Seen (2026-10-17)

- `test-tiers` (D9, 2026-10-07) added a per-package test-count budget: `cmd/lyx/testdata/test-budget.yaml`, guarded by `cmd/lyx/testbudget_test.go`, so that "guards keep the suite from regrowing unseen".
  Its bound: "a budget raise is a one-line edit in the same commit as the tests that need it, so the guard forces a visible, reviewed decision".
- No one judges a raise:
  - the alignment stencil treats the file as bookkeeping that never escalates (`contracts/stencils/loom/loom-template-alignment.md`);
  - the code reviewers are not asked to judge raises;
  - the plan review's test-economy lens (`internal/burlerengine/template.yaml`) only checks that a new test is budgeted;
  - the guard's failure names the value to set the row to, so an agent raises it and moves on.
- Cost: the file changed in 114 of 302 commits on main since 2026-10-01; 11 of 34 plans had two or more cards editing it, the largest false scheduling dependency in `docs/research/parallel-batches-2026-10-17/`; every card adding tests must list it under `Edit:` (CLAUDE.md "Test budget").
- What keeps test count down in practice is `PATTERN-test-economy`, enforced in review.

## Operator decision (2026-10-17)

The budget has no purpose: remove it.

## Idea

- Remove `cmd/lyx/testbudget_test.go`, `cmd/lyx/testdata/test-budget.yaml`, the "Test budget" section of `pattern/PATTERN-test-speed.md`, the "Test budget" section of `CLAUDE.md`, and the budget mentions in the alignment stencil and the burler test-economy lens.
- Watch suite growth instead with a recurring test-time audit on the board, measured with `cmd/testtiming` against `docs/benchmarks/test-suite-timing.md`, the way `glyph-audit-recurring` tracks glyph use.

### Orch recommendation

Run it as a darn task, since it only deletes; the recurring audit is a board note, not code.
