# Research: three proposals for keeping per-test coverage maps fresh

Adds a research report under `docs/research/speed/testmap-upkeep-2026-11-11/` and links it from `docs/research/README.md`.
No code changes, no design decision, no test change.

## Why

The prime dobby rebuilds a stale coverage map whole, one process per test, and holds a gate slot while it does.
The operator finds that too large a job to repeat.
The board task `testmap-upkeep` asked whether a smarter upkeep exists, and asked three agents to answer independently: two Fable 5.2 and one Opus 5.5, each at high effort, none seeing another's answer.
No Codex agent took part, since Codex does not run on OSL-1046.

## What lands

- `proposal-fable-a.md`, `proposal-fable-b.md` and `proposal-opus.md`: the three proposals, unedited.
- `report.md`: the comparison.
  It lists where the proposals agree, a side-by-side table, where they differ, a verdict on each of the six starting ideas of the board note, what each proposal asks of the gate-scope tasks `T4`, `T14` and `T15`, the claims checked against the code, the measurements the proposals ask for, and the open questions for a design task.

## What a reviewer should know

- The report is input for a later design task and decides nothing.
- All three proposals agree on a few points:
  the test, not the package, becomes the unit of staleness;
  the grain is the function, not the block;
  one classifier serves both the gate and upkeep;
  the age rebuild goes;
  upkeep never makes a gate wait.
- They differ on where the staleness verdict lives, how upkeep yields a slot, whether it measures in the prime worktree or a pinned checkout, and which tests it measures.
- No agent ran a test or a build, so every duration in a proposal is an estimate or a figure read from an earlier record.
  The estimates of a first full map set differ by a factor of about four between two proposals.

## To check by hand

- The report's claims were checked against the code at commit `4ae8978ec`, with the results in its table.
  The run-record figures of the Opus proposal, the `internal/fabricengine` timings of proposal B, and the weekly sums and non-body counts that all three take from the Sketch of `gate-scope-by-coverage-map` were not checked.
- The six measurements in the report's "Measurements the proposals ask for" section were not run.
  They settle the open questions about build cost and idle time.
