# Measure where a Webster run spends its time, from recorded runs

`webster-time-analysis` · Task · research, webster · done

## Why (operator, 2026-11-08)

The operator wants Webster faster.
Parallel batches are known to buy little (`docs/research/webster/parallel-batches-2026-10-17/`: about 1.3x to 1.6x at best on mid-size plans, nothing on small ones), and Webster stays serial.
No one has measured where a Webster run's time goes today, so the levers are guesses.

## What to measure

From the records the hub already keeps, and no new runs:

- `lym report timing --since 2026-10-25 --format json`, and the same `--by week`, from the prime worktree.
- Each Webster run's own records in the window: its state, partition, batch reports and logs, wherever `internal/websterengine` and the run records keep them; find them by reading the code.

No agent, no test and no gate run: this reads records and code only.

For every Webster run in the window, and summed over them:

- Wall time of the Webster phase, split into: Master start-up, partition and batchifier, each batch's fork from start to report, the card gate and verify of each batch, fix rounds, recoveries, the plan-level verify, and waits for a gate slot or the operator; what is left over is overhead between those.
- Cards per run, batches per run, cards per batch, and how many batches ran a fix round or a recovery.
- Of a fork's time, how much is the agent working and how much is it waiting on a gate or test it started.

## The batcher question

For each recorded plan in the window, form the batches a grouping by the card dependency graph would give (edges from the cards' Targets and Uses, as `CheckBatchOrder` in `internal/websterengine/sequence.go` derives them, under the same cost budget as `internal/batcher/cost.go`), offline in a throwaway test or tool that is not committed, and report the batch count against the recorded one.
This sizes the note `batcher-dag-order`.

## Report

Write `docs/research/webster/webster-time-2026-11-08.md`:

- A table per run and a summary of the split above, as shares of Webster's wall time, slowest parts first.
- The batch-count comparison for the dependency-graph grouping.
- The three to five levers that would save the most time, each with its estimated saving in minutes per run and the evidence for it.

Commit the report and no code.
