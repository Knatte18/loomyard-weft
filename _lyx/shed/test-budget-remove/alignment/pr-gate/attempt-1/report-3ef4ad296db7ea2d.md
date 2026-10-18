---
verdict: APPROVED
---

## Alignment at pr-gate — test-budget-remove

Measured the diff of pull request 721 at `1508cde35` against `main` (one commit, `Remove the per-package test budget`) against board entry `test-budget-remove`.
The decision record has no `Added after Discussion` entries, so the board body is the measure unchanged.

### Board entry points, each met

- `cmd/lyx/testbudget_test.go` and `cmd/lyx/testdata/test-budget.yaml` are deleted.
- The `## Test budget` section of `pattern/PATTERN-test-speed.md` is deleted.
- The `## Test budget` section of `CLAUDE.md` is deleted.
- The budget file is dropped from the shared-bookkeeping sentence in `contracts/stencils/loom/loom-template-alignment.md`.
- The budget clause is dropped from the `test-economy` lens in `internal/burlerengine/template.yaml`.
- The recurring test-time audit is on the board as the `kind: note` entry `test-time-audit-recurring`, naming `cmd/testtiming` and `docs/benchmarks/test-suite-timing.md`, modelled on `glyph-audit-recurring` as the entry's Idea says.

`git grep` at `1508cde35` finds no mention of `test-budget`, `testbudget` or "test budget" outside `docs/research/` and `docs/benchmarks/`, which the record leaves as written records of past runs.
The diff holds no change beyond the six files above.

### Open risks

The record's one open risk, an unfound mention of the budget file, names no parent correction; the grep above finds none.

### Overlap

`lyx loom peers` lists no peer runs and no parent commits (`own_row` 22), so there is no overlap and no landing order to give.
No other board entry touches this task: `timing-report-darn` and `xdr-git-test-file-burst` concern timing reports and git test fixtures and never mention the budget.

### Superseded

No other board entry is made obsolete by this run.
