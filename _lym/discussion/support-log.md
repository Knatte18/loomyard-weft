## Interview

A darn run holds no interview; the scope is read from the board entry and a coarse reading of `docs/benchmarks/test-suite-timing.md` and `cmd/testtiming/main.go`.

## Rejected alternatives

- Rewriting the older Tier 1 and Tier 2 rows to the new tags: the entry says the older rows stay as written.
- Measuring `tier3` and the LLM tier: the entry excludes both.
- Filing prune tasks for the candidates: the entry says the writer files none and a darn run's board refuses a task create.
- A wider sample than 3 warm runs: the entry says exactly 3, matching the file's method.

## Review rounds

_No rounds yet._

## Question ledger

- Q1, resolved by self-pick: which previous after-state is the baseline. The `test-tiers` After state (2026-10-07) is the latest in the file.
- Q2, resolved by self-pick: whether per-package counts exist for the baseline. The file holds per-tier totals only, so D6 falls back to totals when no per-package baseline is found.
