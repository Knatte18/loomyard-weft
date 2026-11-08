# Measure where a Webster run spends its time

Adds `docs/research/webster/webster-time-2026-11-08.md`, a measurement of where a serial Webster run's time goes, taken from the records the hub already keeps.
No code changes: the report is the only file in the commit, and no agent, test or gate ran for it.

## What the report holds

- **A split of the Webster step** for 14 recorded runs (16 ran in the window, two slept inside the step and are left out of the split),
  as a table per run and a summary of shares, slowest part first.
  Agent work in the forks is 62% of the step, the card gate 13% (the guard step is most of it), the plan-level verify gate 7%,
  the batch gate 5%, and waits for a gate slot 3%.
- **Fork time:** 72.5% agent work, 27.5% waiting on a gate or test the fork started, including slot waits.
- **Batches, fix rounds and recoveries:** 45 of 85 batches ran a fix round, one needed a recovery, one was re-forked,
  and 8 of the 14 runs failed the plan-level verify on its first attempt.
- **The batcher question:** a grouping by the card dependency graph, under the recorded `wide` profile, gives 87 batches against the 95 recorded
  over all 16 plans (85 against 78 for the 14 analysed runs), one batch fewer on 8 plans.
- **Five levers** with an estimated saving in minutes per run and the evidence for each:
  the guard step once per batch, a shorter fork orientation read, `cmd/lym` caught before the plan-level verify,
  less gate-slot contention, and the dependency-graph grouping (`batcher-dag-order`).

## Design decisions

- **Webster step, not timing's `webster` phase.** `lym report timing` files the Code-Review rows, Drift-Verify and Describe under `webster` too,
  so the report measures the span from `Webster -> running` to the next phase change, and says how the two differ.
  A plan-review reroute in the middle of a run ends the span, so that time is not counted.
- **Exact gate time, from the gate run records.** A fork's gate wall and slot wait come from `.lym/gate/runs/*.json`, not from transcript call durations,
  because a gate Bash moves to the background after 180 s runs on past the call.
  Agent work is the remainder of a fork's span, so it is an upper bound.
- **The dependency-graph count is a greedy upper bound on the minimum.**
  A throwaway tool (not committed, and removed) priced cards from the per-card components recorded in each run's partition breakdown.
  It reproduced every recorded batch count and estimate for the contiguous grouping before it grouped by the graph.
- **Two runs left out of the split.** `interactive-recipe` and `cold-gate-fixer` slept for hours inside the step, which the commit times cannot separate from work.
  Both stay in the batcher comparison, which needs only their plans.

## Review notes

- The report is a point-in-time measurement and goes stale when new runs land; it makes no claim to stay true.
- One stall in `creel-cross-hub-build` (27.9 min, a fork that died on an API error) is reported but not made a lever.
- Estimated savings for the orientation lever rest on an assumption the report states (the read is halved); the other four rest on measured quantities.
