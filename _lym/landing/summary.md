# Report fabricengine tier 1 wall and git spawns around e19a2e86

Adds `docs/research/speed/fabricengine-spawns-2026-11-09.md`, a measurement report on `internal/fabricengine` tier 1 at `e19a2e86^` and at `e19a2e86`.
No code or test changed.

## What the report holds

- On this machine the tier 1 wall fell from 47.7 s to 31.5 s and git processes from 61 291 to 58 693, 2 598 fewer.
- The 2 598 are `rev-parse` (1 743), `ls-tree` (452) and `show` (418), the `HEAD` and committed-file reads the commit moved into Go.
- The ten subcommands and the ten call sites started most often after the change, each marked read-only or writing, plus a list of production read-only shapes as input for a next task.

## Design decisions

- Each commit got one run, through `lym gate test --wait --tier 1 ./internal/fabricengine` with the shard runner's one-process form, as the task asks.
  Neither run failed, so none was repeated.
- The runs used this worktree checked out detached at each commit in turn, and the branch was restored afterwards.
- The load differed between the runs (mean one-minute load 19.3 before, 16.6 after), so the report says the wall difference is not all the change's.
- Trace2 holds argv and no stack, so call sites are attributed by reading the source for each argv shape.
  A shape that a test helper also starts is named for both and not split.

## Verification

Both runs passed the tier (1381 and 1392 tests passed, none failed).
The raw trace files and samples stay in the run's scratch directory and are not committed.
