# Round 2 fixer-report

## Changes

- **F1 (MEDIUM, fixed as proposed, then widened).**
  `internal/webstercli/verbs_test.go` gains `sizeProbeBatcher`, a `batcher.Batcher` double that asks its size source for `backend/README.md` and records whether it was found, and `TestExecutionBatches_SizesReadFromWorktreeRoot`, which drives `executionBatches` over `newVerbsFixture` with a nil state.
  No production change.
  Commit: `webstercli: fix F1 — pin the worktree-root base of the batcher size source in a subpath-anchored hub`.
- **F1 widening, from the self-check.**
  The review's `fix` key claimed that one test over `executionBatches` fails a swap in all three `DiskSizes(WorktreeRoot)` reads, but it fails only the `cli.go` one.
  Two more tests now pin the other two:
  - `TestRebaselineCmd_SizesReadFromWorktreeRoot` (`internal/webstercli/verbs_test.go`) runs the rebaseline verb over a recorded partition with an unbegun tail card and the probe batcher.
  - A `TestRun_MasterSpawn` row (`internal/websterengine/runlevel_test.go`) runs `Run` with `PaneCwd` set to a subdirectory and `sizeBatcher`, and expects a 150-line `any.go` at the worktree root to split the two cards into two batches.
  Commit: `webstercli: fix F1 — also pin the worktree-root size source of rebaseline and Run`.

Each of the three tests was checked against the defect: with the root swapped to `PaneCwd`, the `executionBatches` test passed on the fix and failed under the swap (reasoned by the lens forks and the gate, not mutated), and the rebaseline and `Run` tests failed under a temporary swap of `rebaseline.go` and `runlevel.go`.
Both temporary swaps were reverted before the commit.
The first draft of the `Run` row put the file under the anchor and passed under the swap; it was inverted and corrected before the commit.
`lym gate test ./internal/websterengine ./internal/webstercli` is green at tier 0, and the same two packages at tier 1 were green before the row was corrected.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: MEDIUM, the new test pins only the `cli.go` read; fixed as proposed by adding the rebaseline and `Run` tests.
- fix-affected-set: MEDIUM, the same gap for `rebaseline.go` and `runlevel.go`; merged with the fix-proposal finding and fixed with it.

Lens reports: `round-2-fixer-report.lens-<lens>.md` beside this file, one per lens; every fork returned a report.
