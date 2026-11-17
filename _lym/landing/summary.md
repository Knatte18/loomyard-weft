# Hand a run over to the new build even with uncommitted paths

A landing can install new board copies of the stencils while a run is mid-step.
The old driver then kept running its old binary against stencils whose includes it lacks.
Seen on `launch-manifest-parity`: `gatefix: fill the gatefix-body-plan prompt: stencil: include "engine-absolute-paths" has no entry in the includes`.

The cause was in `handOver` in `internal/sheddriver/drive.go`.
It saw that the on-disk build had changed, but it spawned no successor while the plan writer's files in `_lym/plan/` were uncommitted.
So the old binary rendered the new copies and failed the run.

`handOver` now hands over whatever the tree holds.
It no longer checks for uncommitted paths, and the `DirtyPaths` seam field and its wiring helper `dirtyPathsOutsideRecords` in `internal/loomcli/drivewiring.go` are removed.
The successor driver of the on-disk build re-steps over the uncommitted output.

Closes nothing: the board entry names no issue.

## Design decisions

- A step of the old build that fails midway leaves its producer's output uncommitted, and only the on-disk build can re-step it.
  A clean-tree gate therefore blocks exactly the hand-over that is needed.
- Of the three ideas on the board entry, this takes the third in effect: the old binary stops running once the build has changed, rather than rendering embedded stencils or waiting for the producer to commit.
- A failed successor spawn still leaves the run parked with its report.
  A watching batten re-steps it only over a tree with no uncommitted paths outside the run's records, so a failed spawn over uncommitted paths waits for the parent.
- The hand-over doc in `internal/loomcli/doc.go` and the `handOver` comment are updated to match.

## To check

- The hand-over test table in `internal/sheddriver/drive_test.go` lost the dirty-tree row and the `dirty` field; no test was run by hand beyond that.
- Check by hand that a hand-over over uncommitted paths in `_lym/plan/` lets the successor re-step cleanly without losing the producer's files.
