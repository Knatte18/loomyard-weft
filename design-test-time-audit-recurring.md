# Recurring test-suite time and growth audit

`test-time-audit-recurring` · Note · audit

## What this is

A recurring audit of test-suite growth, run with `go run ./cmd/testtiming` (Tier 1) and `go run ./cmd/testtiming -full` (Tier 2), and compared against the latest after-state in `docs/benchmarks/test-suite-timing.md`.
It replaces the per-package test-count budget that `test-budget-remove` deleted (operator decision 2026-10-17): count is kept down by `PATTERN-test-economy` in review, and this audit watches the wall time and the per-package test counts that review cannot see in aggregate.
Each audit appends its numbers as a new section of `docs/benchmarks/test-suite-timing.md`, under the method that file states.

## What to look at

- Wall time per tier against the previous section, and the packages that moved most.
- Per-package test counts from `cmd/testtiming`'s `TESTS` column, and which packages grew since the previous audit.
- A growth with no matching new behavior is a candidate for a prune task under `PATTERN-test-economy`.

## Orch recommendation, not a decision

Run the audit every two weeks or after about 50 landed tasks, whichever comes first.
A cheap model (Sonnet) runs the measurement; the orch writes the section and files a prune task when a package grew without cause.
