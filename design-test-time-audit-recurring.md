# Recurring test-suite time and growth audit

`test-time-audit-recurring` · Task · research, gate · running · Darn-Write

## What this is

A recurring audit of test-suite growth, run with `go run ./cmd/testtiming` and its `-tags` flag per tier (its `main.go` names the tiers), and compared against the latest after-state in `docs/benchmarks/test-suite-timing.md`.
It replaces the per-package test-count budget that `test-budget-remove` deleted (operator decision 2026-10-17): count is kept down in review by `PATTERN-test-speed`, which absorbed `PATTERN-test-economy` in `04aafa88c`, and this audit watches the wall time and the per-package test counts that review cannot see in aggregate.
Each audit appends its numbers as a new section of `docs/benchmarks/test-suite-timing.md`, under the method that file states.

## What to look at

- Wall time per tier against the previous section, and the packages that moved most.
- Per-package test counts from `cmd/testtiming`'s `TESTS` column, and which packages grew since the previous audit.
- A growth with no matching new behavior is a candidate for a prune task under `PATTERN-test-speed`.

## This run (operator, 2026-10-28)

The first audit runs now, on Neve while no other run is in flight, so the machine is quiet.
- Tiers `tier0`, `tier1` and `tier2`, each as the median of exactly 3 warm runs after `go build ./...`, as the file's method states; no `tier3` (tmux) and no LLM tier.
- The file still names the tiers Tier 1 and Tier 2, from before the gate's tier tags; the new section names the tags and says how they map to the older columns, and the older rows stay as written.
- Before the measurement, check that nothing else runs on the machine (`lym tender status`, `lym gate status`); a busy machine is recorded in the section, not hidden.
- When lym refuses a `go run ./cmd/testtiming` call under the agent's run rules, take the route the refusal names and record it in the section.
- The section names the per-package growth since the last recorded after-state and the candidates for a prune task; the writer files no task, and each candidate stays a line in the section.

## Orch recommendation, not a decision

Run the audit every two weeks or after about 50 landed tasks, whichever comes first.
A cheap model (Sonnet) runs the measurement; the orch writes the section and files a prune task when a package grew without cause.
